@RequestMapping:  定义在类上，统一访问路径



@RequestBody 定义在方法中的形参列表前 可以把请求列表封装成一个对象，如果是多个单个的消息，应用RequestParam

只用于接收 **整个请求体**，绑定到一个对象。



@RequestParam 处理多个单个的消息，也可不加，不加就是默认



```
@Autowired spring的注解，一般在service和controller中定义的控制实例交给spring控制
```

