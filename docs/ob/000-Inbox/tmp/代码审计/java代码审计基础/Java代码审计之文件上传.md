## 基本概况

访问url为[http://localhost:8080/file/any](http://localhost:8080/file/any)-
直接对上传的文件保存在了指定路径下

    @PostMapping("/upload")
    public String singleFileUpload(@RequestParam("file") MultipartFile file,RedirectAttributes redirectAttributes) {
    	if (file.isEmpty()) {
    		// 赋值给uploadStatus.html里的动态参数message
    		redirectAttributes.addFlashAttribute("message", "Please select a file to upload");
    		return "redirect:/file/status";
    	}
    

没有任何的后缀名及内容过滤，可以上传任意的恶意文件。-
在这里上传目录在/tmp下，同时文件的写入时用的Files.write(path,bytes)，这里的path就是保存的路径，由于是保存的目录/tmp直接拼接了文件名，就可以在文件名中利用../来达到目录穿越的目的，从而将任意文件保存在任意目录下。-
代码跟踪-
1、因为没有任何过滤，所以可以上传任意后缀-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220907%2F1662559458_6318a4e23f83ad7dadd4e.png%21small)-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220907%2F1662559484_6318a4fcceb716517a1e0.png%21small)-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220907%2F1662559501_6318a50d9a8d339f72203.png%21small)-
2、也可以进行路径遍历，上传到上一层目录-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220907%2F1662559515_6318a51b60c23178d3df3.png%21small)

**pic**-
限制只能上传图片，同时进行了多重验证。-
对文件后缀名进行白名单限制，只能为白名单中的图片后缀名。-
AAA.JSP.

## 后缀名过滤

    // 判断文件后缀名是否在白名单内
    String[] picSuffixList = {".jpg", ".png", ".jpeg", ".gif", ".bmp", ".ico"};
    boolean suffixFlag = false;
    for (String white_suffix : picSuffixList) {
    	if (Suffix.toLowerCase().equals(white_suffix)) {
    		suffixFlag = true;
    		break;
    	}
    }
    

## MIME过滤

对MIME类型进行了黑名单限制，不过这个可以进行抓包修改绕过

    // 判断MIME类型是否在黑名单内 校验2
    String[] mimeTypeBlackList = {
    		"text/html",
    		"text/javascript",
    		"application/javascript",
    		"application/ecmascript",
    		"text/xml",
    		"application/xml"
    };
    for (String blackMimeType : mimeTypeBlackList) {
    	// 用contains是为了防止text/html;charset=UTF-8绕过
    	if (SecurityUtil.replaceSpecialStr(mimeType).toLowerCase().contains(blackMimeType)) {
    		logger.error("[-] Mime type error: " + mimeType);
    		//deleteFile(filePath);
    		return "Upload failed. Illeagl picture.";
    	}
    }
    

## 路径穿越过滤

文件保存的时候路径是通过 来获取，Path path =excelFile.toPath();就避免了路径穿越的实现

    File excelFile = convert(multifile);//文件名字做了uuid处理
    String filePath = excelFile.getPath();
    // 判断文件内容是否是图片 校验3
    boolean isImageFlag = isImage(excelFile);
    if (!isImageFlag) {
    	logger.error("[-] File is not Image");
    	deleteFile(filePath);
    	return "Upload failed. Illeagl picture.";
    }
    

## 对上传的文件过滤

    判断上传的文件是否为图片，通过ImageIO.read对文件进行读取来判断
    private static boolean isImage(File file) throws
    IOException {
    BufferedImage bi = ImageIO.read(file);
    return bi != null;
    }
    

## 代码跟踪

### 上传txt文件

![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220907%2F1662559535_6318a52fe3f4fe86f1177.png%21small)-
比较后发现不一样，进入if (!suffixFlag)中-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220907%2F1662559553_6318a54106ddc1e3dbb7f.png%21small)-
最后输出Upload failed. Illeagl picture.在页面上-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220907%2F1662559565_6318a54d75564a3df1672.png%21small)

### 上传做了假的png图片

![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220907%2F1662559577_6318a5591583ce2b4b74e.png%21small)-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220907%2F1662559587_6318a5631ed0e71a1a946.png%21small)-
绕过了后缀和MIME检测，上传上去了，但在最后还判断了服务器上的图片内容，不符合删除-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220907%2F1662559615_6318a57fc85478384a54b.png%21small)