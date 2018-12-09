+ 第一种：prototype原型bean循环依赖
> 对于原型bean的初始化过程中不论是通过构造器参数循环依赖

> 还是通过setXxx方法产生循环依赖，Spring都会直接报错处理。


##### AbstractBeanFactory.doGetBean()方法：

    if (isPrototypeCurrentlyInCreation(beanName)) {
    		throw new BeanCurrentlyInCreationException(beanName);
    	}

    protected boolean isPrototypeCurrentlyInCreation(String beanName) {
    		Object curVal = this.prototypesCurrentlyInCreation.get();
    		return (curVal != null &&
    		(curVal.equals(beanName) ||
    		(curVal instanceof Set && ((Set<?>) curVal).contains(beanName))));
    	}

>在获取bean之前如果这个原型bean正在被创建则直接抛出异常。

>原型bean在创建之前会进行标记这个beanName正在被创建，

>等创建结束之后会删除标记

    try {
    		//创建原型bean之前添加标记
    		beforePrototypeCreation(beanName);
    		//创建原型bean
    		prototypeInstance = createBean(beanName, mbd, args);
    	}
    	finally {
    		//创建原型bean之后删除标记
    		afterPrototypeCreation(beanName);
    	}

#### 总结：简单来说Spring不支持原型bean的循环依赖。

+ 第二种：单例bean 构造器参数循环依赖
>对于单例bean通过构造器参数进行循环依赖时，Spring也是通过抛出异常进行处理的，接下来我们分析一下其是如何实现。假设这里有两个类ClassA 和 ClassB，两个类通过构造器进行循环依赖。
1、Spring创建ClassA时首先会调用beforeSingletonCreation(beanName)，判断单例bean是否正在被创建，如果正在被创建则报错，如果没有被创建则做标记

    protected void beforeSingletonCreation(String beanName) {
    		//singletonsCurrentlyInCreation是一个集合，不可重复，add返回true则表明没有创建，返回false则说明bean在创建
    		if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
    			throw new BeanCurrentlyInCreationException(beanName);
    		}
    	}

>此时刚开始创建ClassA，因此不会报异常错误，当初始化ClassA的构造器时需要用到ClassB的实例。

2、Spring从容器中无法获取ClassB的实例，接下来Spring按照步骤1创建ClassB的实例，在构造ClassB的构造函数时需要用到ClassA的实例。
3、Spring还是首先尝试从容器中获取ClassA的实例，由于第1步还没有执行结束，此时还没有ClassA实例，Spring容器认为就需要初始化ClassA了，重复第1步，
但是ClassA一开始就已经进行了第1步，并且做标记bean正在创建，当再次进入第一步时就会抛出异常错误。

##### 总结：Spring在创建构造器循环依赖时其实就是循环初始化操作 A-> B -> A  当A要被初始化第二次时就直接抛出异常。

+ 第三种 ：单例bean通过setXxx或者@Autowired进行循环依赖
>Spring通过setXxx或者@Autowired方法解决循环依赖其实是通过提前暴露一个ObjectFactory对象来完成的，简单来说ClassA在调用构造器完成对象初始化之后，在调用ClassA的setClassB方法之前就把ClassA实例化的对象通过ObjectFactory提前暴露到Spring容器中。
1、Spring容器初始化ClassA通过构造器初始化对象后提前暴露到Spring容器。

    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
    				isSingletonCurrentlyInCreation(beanName));
    		if (earlySingletonExposure) {
    			if (logger.isDebugEnabled()) {
    				logger.debug("Eagerly caching bean '" + beanName +
    						"' to allow for resolving potential circular references");
    			}
    			//将初始化后的对象提前已ObjectFactory对象注入到容器中
    			addSingletonFactory(beanName, new ObjectFactory<Object>() {
    				@Override
    				public Object getObject() throws BeansException {
    					return getEarlyBeanReference(beanName, mbd, bean);
    				}
    			});
    		}

2、ClassA调用setClassB方法，Spring首先尝试从容器中获取ClassB，此时ClassB不存在Spring容器中。
3、Spring容器初始化ClassB，同时也会将ClassB提前暴露到Spring容器中
4、ClassB调用setClassA方法，Spring从容器中获取ClassA ，因为第一步中已经提前暴露了ClassA，因此可以获取到ClassA实例
5、ClassA通过spring容器获取到ClassB，完成了对象初始化操作。
6、这样ClassA和ClassB都完成了对象初始化操作，解决了循环依赖问题。


##### 总结：简单来说就是对象通过构造函数初始化之后就暴露到容器中，这样就不会存在循环初始化对象的情况了。