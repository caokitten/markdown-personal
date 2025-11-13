### 事务

- ```java
  @Transactional
  //注解在方法上,调用方法时自动启用事务,调用结束时提交
  //默认情况下,只有RuntimeException运行时异常才会使事务回滚
  @Transactional(rollbackFor = {Exception.class})
  //使用rollbackFor属性,指定触发事务回滚的异常类型
  //只有一个指定异常类型时,{}可以省略
  
  /**事务的传播行为:
  *指的就是在A方法运行的时候，首先会开启一个事务，在A方法当中又调用了B方法， B方法自身也具有事务，那么B方法在运行的时候，到底是加入到A方法的事务当中来，还是B方法在运行的时候新建一个事务？这个就涉及到了事务的传播行为。
  */
  @Transactional(propagation = Propagation.REQUIRES_NEW)
  //默认情况下,上述行为,B方法不会创建新的事务,而是使用同一个事务,B出现错误立刻回滚,剩余代码都不会执行
  //如上propagation属性,REQUIRES_NEW值可以在每次调用新方法时开启一个新的事务
  ```

- 事务的特性(ACID)
  - 原子性
  - 一致性
  - 隔离性
  - 持久性

### 文件上传

- 本地存储

  ```html
  <form action="/upload" method="post" enctype="multipart/form-data">
          姓名: <input type="text" name="username"><br>
          年龄: <input type="text" name="age"><br>
          头像: <input type="file" name="file"><br>
          <input type="submit" value="提交">
      </form>
  <!--
  	 1.请求方式必须为post请求
  	 2.表单类型必须为file
  	 3.编码类型必须为multipart/form-data
  -->
  ```

  

  ```java
  MultipartFile
  //由spring提供的API,用于存储管理接收到的文件
  //表单名与参数名不一致:
  @RequestParam("file")
  //参数注解,用于参数绑定
      
  @Slf4j
  @RestController
  public class UploadController {
  
      private static final String UPLOAD_PATH = "D:\\upload\\";
  
      /**
       * 上传文件 - 参数名file
       */
      @PostMapping("/upload")
      public Result upload(String username, Integer age , MultipartFile file) throws Exception {
          log.info("上传文件：{}, {}, {}", username, age, file);
          if(!file.isEmpty()){
              //获取文件名
              String fileName = file.getOriginalFilename();
              //分割出文件后缀名
              String suffix = fileName.substring(fileName.lastIndexOf("."));
              //拼接文件名
              String uniqueFileName = UUID.randomUUID().toString().replace(",","") + suffix;
              //拼接完整路径
              File dest = new File(UPLOAD_PATH + uniqueFileName);
  
              if(!dest.getParentFile().exists()){
                  //若目录不存在,创建目录
                  dest.getParentFile().mkdirs();
              }
  
              //保存文件
              file.transferTo(dest);
          }
          return Result.success();
      }
  
  }
  ```

  在 SpringBoot 中，文件上传时默认单个文件最大大小为 1M
  
  那么如果需要上传大文件，可以在 `application.properties` 进行如下配置：
  
  ```yml
  spring：
    servlet:
      multipart:
        max-file-size: 10MB
        max-request-size: 100MB
  ```

### 云服务: OOS 云对象存储

- 开通云服务: 略

- 配置 AccessKey

  ```sql
  set OSS_ACCESS_KEY_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  set OSS_ACCESS_KEY_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  #配置系统环境变量
  setx OSS_ACCESS_KEY_ID "%OSS_ACCESS_KEY_ID%"
  setx OSS_ACCESS_KEY_SECRET "%OSS_ACCESS_KEY_SECRET%"
  #设置为永久保存
  echo %OSS_ACCESS_KEY_ID%
  echo %OSS_ACCESS_KEY_SECRET%
  #控制台输出变量
  ```

- 链接

  ```java
  // 配置OSS中bucket对应的域名
  String endpoint = "https://oss-cn-beijing.aliyuncs.com";
  // 在环境变量中获取访问凭证
  EnvironmentVariableCredentialsProvider credentialsProvider = CredentialsProviderFactory.newEnvironmentVariableCredentialsProvider();
  // bucket名称
  String bucketName = "java-ai";
  // 填写Object完整路径，例如exampledir/exampleobject.txt。Object完整路径中不能包含Bucket名称。
  String objectName = "001.jpg";
  // bucket所在地域
  String region = "cn-beijing";
  // 创建OSSClient实例
  ClientBuilderConfiguration clientBuilderConfiguration = new ClientBuilderConfiguration();
  // 配置云存储服务客户端使用V4签名协议
  clientBuilderConfiguration.setSignatureVersion(SignVersion.V4);
  // 配置信息
  OSS ossClient = OSSClientBuilder.create()
              .endpoint(endpoint)
              .credentialsProvider(credentialsProvider)
              .clientConfiguration(clientBuilderConfiguration)
              .region(region)
              .build();
  // 使用
  try {
              File file = new File("C:\\Users\\deng\\Pictures\\1.jpg");
      //	   读取全部内容至字节数组
              byte[] content = Files.readAllBytes(file.toPath());
  	//	   将数据存入bucket
              ossClient.putObject(bucketName, objectName, new ByteArrayInputStream(content));
          } catch (OSSException oe) {
              System.out.println("Caught an OSSException, which means your request made it to OSS, "
                      + "but was rejected with an error response for some reason.");
              System.out.println("Error Message:" + oe.getErrorMessage());
              System.out.println("Error Code:" + oe.getErrorCode());
              System.out.println("Request ID:" + oe.getRequestId());
              System.out.println("Host ID:" + oe.getHostId());
          } catch (ClientException ce) {
              System.out.println("Caught an ClientException, which means the client encountered "
                      + "a serious internal problem while trying to communicate with OSS, "
                      + "such as not being able to access the network.");
              System.out.println("Error Message:" + ce.getMessage());
          } finally {
              if (ossClient != null) {
                  ossClient.shutdown();
              }
          }
  ```
  
- 集成

  - 封装

    ```java
    @Component
    public class AliyunOSSOperator {
    
        private String endpoint = "https://oss-cn-beijing.aliyuncs.com";
        private String bucketName = "java-ai";
        private String region = "cn-beijing";
    
        public String upload(byte[] content, String originalFilename) throws Exception {
            // 从环境变量中获取访问凭证。运行本代码示例之前，请确保已设置环境变量OSS_ACCESS_KEY_ID和OSS_ACCESS_KEY_SECRET。
            EnvironmentVariableCredentialsProvider credentialsProvider = CredentialsProviderFactory.newEnvironmentVariableCredentialsProvider();
    
            // 填写Object完整路径，例如202406/1.png。Object完整路径中不能包含Bucket名称。
            //获取当前系统日期的字符串,格式为 yyyy/MM
            String dir = LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy/MM"));
            //生成一个新的不重复的文件名
            String newFileName = UUID.randomUUID() + originalFilename.substring(originalFilename.lastIndexOf("."));
            String objectName = dir + "/" + newFileName;
    
            // 创建OSSClient实例。
            ClientBuilderConfiguration clientBuilderConfiguration = new ClientBuilderConfiguration();
            clientBuilderConfiguration.setSignatureVersion(SignVersion.V4);
            OSS ossClient = OSSClientBuilder.create()
                    .endpoint(endpoint)
                    .credentialsProvider(credentialsProvider)
                    .clientConfiguration(clientBuilderConfiguration)
                    .region(region)
                    .build();
            try {
                ossClient.putObject(bucketName, objectName, new ByteArrayInputStream(content));
            } finally {
                ossClient.shutdown();
            }
            return endpoint.split("//")[0] + "//" + bucketName + "." + endpoint.split("//")[1] + "/" + objectName;
        }
    }
    ```

  - Controller 层

    ```java
    @Slf4j
    @RestController
    public class UploadController {
        
        @Autowired
        private AliyunOSSOperator aliyunOSSOperator;
    
        @PostMapping("/upload")
        public Result upload(MultipartFile file) throws Exception {
            log.info("上传文件：{}", file);
            if (!file.isEmpty()) {
                // 生成唯一文件名
                String originalFilename = file.getOriginalFilename();
                String extName = originalFilename.substring(originalFilename.lastIndexOf("."));
                String uniqueFileName = UUID.randomUUID().toString().replace("-", "") + extName;
                // 上传文件
                String url = aliyunOSSOperator.upload(file.getBytes(), uniqueFileName);
                return Result.success(url);
            }
            return Result.error("上传失败");
        }
    }
    ```

- 配置参数调整至配置文件

  ```yml
  #阿里云OSS
  aliyun:
    oss:
      endpoint: https://oss-cn-beijing.aliyuncs.com
      bucketName: java-ai
      region: cn-beijing
  ```

  ```java
  @Component
  public class AliyunOSSOperator {
  
      //方式一: 通过@Value注解一个属性一个属性的注入
      @Value("${aliyun.oss.endpoint}")
      private String endpoint;
      
      @Value("${aliyun.oss.bucketName}")
      private String bucketName;
      
      @Value("${aliyun.oss.region}")
      private String region;
  
      public String upload(byte[] content, String originalFilename) throws Exception {
          // 从环境变量中获取访问凭证。运行本代码示例之前，请确保已设置环境变量OSS_ACCESS_KEY_ID和OSS_ACCESS_KEY_SECRET。
          EnvironmentVariableCredentialsProvider credentialsProvider = CredentialsProviderFactory.newEnvironmentVariableCredentialsProvider();
  
          // 填写Object完整路径，例如2024/06/1.png。Object完整路径中不能包含Bucket名称。
          //获取当前系统日期的字符串,格式为 yyyy/MM
          String dir = LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy/MM"));
          //生成一个新的不重复的文件名
          String newFileName = UUID.randomUUID() + originalFilename.substring(originalFilename.lastIndexOf("."));
          String objectName = dir + "/" + newFileName;
  
          // 创建OSSClient实例。
          ClientBuilderConfiguration clientBuilderConfiguration = new ClientBuilderConfiguration();
          clientBuilderConfiguration.setSignatureVersion(SignVersion.V4);
          OSS ossClient = OSSClientBuilder.create()
                  .endpoint(endpoint)
                  .credentialsProvider(credentialsProvider)
                  .clientConfiguration(clientBuilderConfiguration)
                  .region(region)
                  .build();
  
          try {
              ossClient.putObject(bucketName, objectName, new ByteArrayInputStream(content));
          } finally {
              ossClient.shutdown();
          }
  
          return endpoint.split("//")[0] + "//" + bucketName + "." + endpoint.split("//")[1] + "/" + objectName;
      }
  
  }
  ```

- 简化配置注入(将配置文件中的配置项的值自动注入到对象的属性中)

  ```java
  // 创建实现类,其属性名需要与配置文件中的属性名一致,同时需要提供getter与setter方法
  // 交由IOC容器管理
  // 添加@ConfigurationProperties注解,通过prefix属性指定配置参数项的前缀
  @Data
  @Component
  @ConfigurationProperties(prefix = "aliyun.oss")
  public class AliyunOSSProperties {
      private String endpoint;
      private String bucketName;
      private String region;
  }
  ```

  ```java
  @Component
  public class AliyunOSSOperator {
      @Autowired
      private AliyunOSSProperties aliyunOSSProperties;
  
      public String upload(byte[] content, String originalFilename) throws Exception {
          String endpoint = aliyunOSSProperties.getEndpoint();
          String bucketName = aliyunOSSProperties.getBucketName();
          String region = aliyunOSSProperties.getRegion();
  
          // 从环境变量中获取访问凭证。运行本代码示例之前，请确保已设置环境变量OSS_ACCESS_KEY_ID和OSS_ACCESS_KEY_SECRET。
          EnvironmentVariableCredentialsProvider credentialsProvider = CredentialsProviderFactory.newEnvironmentVariableCredentialsProvider();
  
          // 填写Object完整路径，例如2024/06/1.png。Object完整路径中不能包含Bucket名称。
          //获取当前系统日期的字符串,格式为 yyyy/MM
          String dir = LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy/MM"));
          //生成一个新的不重复的文件名
          String newFileName = UUID.randomUUID() + originalFilename.substring(originalFilename.lastIndexOf("."));
          String objectName = dir + "/" + newFileName;
  
          // 创建OSSClient实例。
          ClientBuilderConfiguration clientBuilderConfiguration = new ClientBuilderConfiguration();
          clientBuilderConfiguration.setSignatureVersion(SignVersion.V4);
          OSS ossClient = OSSClientBuilder.create()
                  .endpoint(endpoint)
                  .credentialsProvider(credentialsProvider)
                  .clientConfiguration(clientBuilderConfiguration)
                  .region(region)
                  .build();
  
          try {
              ossClient.putObject(bucketName, objectName, new ByteArrayInputStream(content));
          } finally {
              ossClient.shutdown();
          }
  
          return endpoint.split("//")[0] + "//" + bucketName + "." + endpoint.split("//")[1] + "/" + objectName;
      }
  }
  ```

  