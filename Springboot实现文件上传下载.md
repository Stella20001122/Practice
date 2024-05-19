# Springboot实现文件上传下载

###### --0514--SXJ

###### 项目：testfile

1. 新建springboot项目，添加web依赖
2. 建立control包，新建上传控制类

```
package com.file.control;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;
import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

@RestController
//@Controller 
public class FileUploadController {
    private static final String UPLOAD_DIR = "D:/uploads/";

    @PostMapping("/upload")
    //@GetMapping("/upload")
    public ResponseEntity<?> uploadFile(@RequestParam("file") MultipartFile file) {
    //public String uploadFile(@RequestParam("file") MultipartFile file) {	
        System.out.println("执行到这里了");
    	if (file.isEmpty()) {
            return ResponseEntity.badRequest().body("请选择一个文件上传");
        }

        try {
            byte[] bytes = file.getBytes();
            Path path = Paths.get(UPLOAD_DIR + file.getOriginalFilename());
            Files.write(path, bytes);
            return ResponseEntity.ok("success upload!--" + file.getOriginalFilename());
        } catch (IOException e) {
            e.printStackTrace();
            return ResponseEntity.status(500).body("failed upload!--" + e.getMessage());
        }
    }
//    @GetMapping("/upload")
//    public String uploadFile() {
//    	System.out.println("执行到这里了");
//    	return("success");
//    }
}
```

设置application.properties

```
# application.properties
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=50MB
```

3. 使用postman测试上传

注意，<u>除了原本的请求头以外</u>，加一个Content-Type设置为multipart/form-data。

Body设置为form-data，设置key为file，上传文件为value，然后send

[在Postman中上传Multipart文件 (baidu.com)](https://cloud.baidu.com/article/3313586)

![](D:\godblessing\forwork\操作记录\figure\springboot+file-1.png)

![](D:\godblessing\forwork\操作记录\figure\springboot+file-2.png)

4. 新建下载控制类

```
package com.file.control;

import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

@RestController
public class FileDownloadController {

    private static final String UPLOAD_DIR = "D:/uploads/";

    @GetMapping("/download/{filename:.+}")
    public ResponseEntity<Resource> downloadFile(@PathVariable String filename) {
        Path filePath = Paths.get(UPLOAD_DIR + filename);
        Resource resource = null;
        try {
            if (Files.exists(filePath)) {
                resource = new FileSystemResource(filePath);
            } else {
                return ResponseEntity.notFound().build();
            }
        } catch (Exception e) {
            e.printStackTrace();
            return ResponseEntity.status(500).body(null);
        }

        // 设置响应头
        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Disposition", "attachment; filename=" + resource.getFilename());
        headers.add("Cache-Control", "no-cache, no-store, must-revalidate");
        headers.add("Pragma", "no-cache");
        headers.add("Expires", "0");

        return ResponseEntity.ok()
                .headers(headers)
                .contentType(MediaType.parseMediaType("application/octet-stream"))
                .body(resource);
    }
}
```

5. 测试下载

记得要把get请求里面的content-type取消，否则会返回500

浏览器输入http://localhost:8080/download/内存模型.png 发现可以下载

![](D:\godblessing\forwork\操作记录\figure\springboot+file-3.png)

![](D:\godblessing\forwork\操作记录\figure\springboot+file-4.png)