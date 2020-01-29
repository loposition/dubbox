# 简介
这个版本是dubbox 迁移到dubbo 过程的过度版本
> 不该应用包名试无法上传到中央仓库的 但是我改了又会带来迁移的麻烦 所以还请自行编译

迁移过程
- 项目clone 并编译

```$xslt
clean install  -Dmaven.test.skip=true
``` 
> 请自行deploy
- 将dubbo 的版本换成 2.8.4-dubbo-adapter 

```
  <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>dubbo</artifactId>
      <version>2.8.4-dubbo-adapter</version>
    </dependency>

```

- 所有dubbox 依赖都升级成功之后就可以引入原版的dubbo了

### 进度
目前已经完成的功能只在走了注册中心的才会生效，因为是通过dubbo url 的版本号去判断dubbox 还是dubbo的 如果静态调用的url 中 没有 dubbo=2.8.4 那么也会失效



##dubbox 直接到 dubbo 的迁移版本问题
dubbox迁移到dubbo的时候会因为dubbox 修改了dubbo request的构成 详情请见DecodeableRpcInvocation
dubbox 的协议解码部分
```
            // NOTICE modified by lishen
                    int argNum = in.readInt();
                    if (argNum >= 0) {
                        if (argNum == 0) {
                            pts = DubboCodec.EMPTY_CLASS_ARRAY;
                            args = DubboCodec.EMPTY_OBJECT_ARRAY;
                        } else {
                            args = new Object[argNum];
                            pts = new Class[argNum];
                            for (int i = 0; i < args.length; i++) {
                                try {
                                    args[i] = in.readObject();
                                    pts[i] = args[i].getClass();
                                } catch (Exception e) {
                                    if (log.isWarnEnabled()) {
                                        log.warn("Decode argument failed: " + e.getMessage(), e);
                                    }
                                }
                            }
                        }
                    } else {
                        String desc = in.readUTF();
                        if (desc.length() == 0) {
                            pts = DubboCodec.EMPTY_CLASS_ARRAY;
                            args = DubboCodec.EMPTY_OBJECT_ARRAY;
                        } else {
                            pts = ReflectUtils.desc2classArray(desc);
                            args = new Object[pts.length];
                            for (int i = 0; i < args.length; i++) {
                                try {
                                    args[i] = in.readObject(pts[i]);
                                } catch (Exception e) {
                                    if (log.isWarnEnabled()) {
                                        log.warn("Decode argument failed: " + e.getMessage(), e);
                                    }
                                }
                            }
                        }
                    }

```
dubbo 的协议解码部分
```

                  String desc = in.readUTF();
                    if (desc.length() == 0) {
                        pts = DubboCodec.EMPTY_CLASS_ARRAY;
                        args = DubboCodec.EMPTY_OBJECT_ARRAY;
                    } else {
                        pts = ReflectUtils.desc2classArray(desc);
                        args = new Object[pts.length];
                        for (int i = 0; i < args.length; i++) {
                            try {
                                args[i] = in.readObject(pts[i]);
                            } catch (Exception e) {
                                if (log.isWarnEnabled()) {
                                    log.warn("Decode argument failed: " + e.getMessage(), e);
                                }
                            }
                        }
                    }
```
可以看出来两个解码的dubbox第一位读的是 一个int  而dubbo 第一位读的是一个字符串

所以dubbox 直接调用任何版本的dubbo 都会直接报出decode error 