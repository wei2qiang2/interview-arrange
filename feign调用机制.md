#### 泛型设计(`fastjson`反序列化成对象泛型设计)

```java
// 方法声明
public <T> T getData(TypeReference<T> typeReference){
        Object data = get("data");
        String jsonString = JSON.toJSONString(data);
        T t = JSON.parseObject(jsonString, typeReference);
        return t;
 }
// 调用
TypeReference<List<SkuHasStockVo>> listTypeReference = new TypeReference<List<SkuHasStockVo>>() {};
List<SkuHasStockVo> skusHasstock = r.getData(listTypeReference);
```



#### `feign`调用源码分析

```java
 /**
  * {@link feign.SynchronousMethodHandler#invoke}
  * 构造请求，将数据转为json
  * 发送请求进行执行,执行成功解码返回结果
  */
public Object invoke(Object[] argv) throws Throwable {
        RequestTemplate template = this.buildTemplateFromArgs.create(argv);
    	// 重试机制
        Retryer retryer = this.retryer.clone();
    	// 循环调用
        while(true) {
            try {
                // 真正的执行远程调用
                return this.executeAndDecode(template);
            } catch (RetryableException var8) {
                RetryableException e = var8;
                try {
                    // 调用重试机制
                    retryer.continueOrPropagate(e);
                } catch (RetryableException var7) {
                    // 当达到最大重试次数或者未开启重试的时候抛出异常，退出循环
                    Throwable cause = var7.getCause();
                    if (this.propagationPolicy == ExceptionPropagationPolicy.UNWRAP && cause != null) {
                        throw cause;
                    }
                    throw var7;
                }
                if (this.logLevel != Level.NONE) {
                    this.logger.logRetry(this.metadata.configKey(), this.logLevel);
                }
            }
        }
    }

```

重试机制相关代码

```java
public interface Retryer extends Cloneable {
    // 默认不重试实现机制，不配置就使用此机制
    Retryer NEVER_RETRY = new Retryer() {
        public void continueOrPropagate(RetryableException e) {
            throw e;
        }
        public Retryer clone() {
            return this;
        }
    };

    void continueOrPropagate(RetryableException var1);

    Retryer clone();

    // 默认的重试机制
    public static class Default implements Retryer {
        private final int maxAttempts;// 最大的重试次数
        private final long period; // 重试时间
        private final long maxPeriod; // 最长重试时间
        int attempt; // 重试次数
        long sleptForMillis;
	    // 默认重试机制的构造方法
        public Default() {
            this(100L, TimeUnit.SECONDS.toMillis(1L), 5);
        }
		
        public Default(long period, long maxPeriod, int maxAttempts) {
            this.period = period;
            this.maxPeriod = maxPeriod;
            this.maxAttempts = maxAttempts;
            this.attempt = 1;
        }

        protected long currentTimeMillis() {
            return System.currentTimeMillis();
        }

        public void continueOrPropagate(RetryableException e) {
            if (this.attempt++ >= this.maxAttempts) {
                throw e;
            } else {
                long interval;
                if (e.retryAfter() != null) {
                    interval = e.retryAfter().getTime() - this.currentTimeMillis();
                    if (interval > this.maxPeriod) {
                        interval = this.maxPeriod;
                    }

                    if (interval < 0L) {
                        return;
                    }
                } else {
                    interval = this.nextMaxInterval();
                }
                try {
                    Thread.sleep(interval);
                } catch (InterruptedException var5) {
                    Thread.currentThread().interrupt();
                    throw e;
                }
                this.sleptForMillis += interval;
            }
        }
        long nextMaxInterval() {
            long interval = (long)((double)this.period * Math.pow(1.5D, (double)(this.attempt - 1)));
            return interval > this.maxPeriod ? this.maxPeriod : interval;
        }

        public Retryer clone() {
            return new Retryer.Default(this.period, this.maxPeriod, this.maxAttempts);
        }
    }
}
```

