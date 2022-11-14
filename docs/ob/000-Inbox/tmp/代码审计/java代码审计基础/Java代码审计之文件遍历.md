## 缺陷代码

    /**
     * http://localhost:8080/path_traversal/vul?filepath=../../../../../etc/passwd
     */
    @GetMapping("/path_traversal/vul")
    public String getImage(String filepath) throws IOException {
    	return getImgBase64(filepath);
    }
    
    
    private String getImgBase64(String imgFile) throws IOException {
    
    	logger.info("Working directory: " + System.getProperty("user.dir"));
    	logger.info("File path: " + imgFile);
    
    	File f = new File(imgFile);
    	if (f.exists() && !f.isDirectory()) {
    		byte[] data = Files.readAllBytes(Paths.get(imgFile));
    		return new String(Base64.encodeBase64(data));
    	} else {
    		return "File doesn't exist or is not a file.";
    	}
    }
    

源码目录的上一层，有test.txt文件，使用../成功读取到上一层目录下的文件内容-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220704%2F1656941448_62c2eb889481343003b39.png%21small)

### 跟踪代码

![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220704%2F1656941456_62c2eb907839106602b2d.png%21small)-
发现只是判断是否有存在，存在的话就以为base64编码形式输出-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220704%2F1656941471_62c2eb9ff261d461c376b.png%21small)

## 修复方法

path\_traversal/sec-
对应的修复后的方法如下，调用了SecurityUtil.pathFilter对用户的输入进行检查。

    @GetMapping("/path_traversal/sec")
    public String getImageSec(String filepath) throws IOException {
    	if (SecurityUtil.pathFilter(filepath) == null) {
    		logger.info("Illegal file path: " + filepath);
    		return "Bad boy. Illegal file path.";
    	}
    	return getImgBase64(filepath);
    }
    

这里对目录穿越的..进行了过滤，避免了目录穿越。只不过这里一个用作图片读取的api也可以读取项目任意文件倒也可以说是算一个小漏洞。

    /**
     * Filter file path to prevent path traversal vulns.
     *
     * @param filepath file path
     * @return illegal file path return null
     */
    public static String pathFilter(String filepath) {
    	String temp = filepath;
    
    	// use while to sovle multi urlencode
    	while (temp.indexOf('%') != -1) {
    		try {
    			temp = URLDecoder.decode(temp, "utf-8");
    		} catch (UnsupportedEncodingException e) {
    			logger.info("Unsupported encoding exception: " + filepath);
    			return null;
    		} catch (Exception e) {
    			logger.info(e.toString());
    			return null;
    		}
    	}
    
    	if (temp.contains("..") || temp.charAt(0) == '/') {
    		return null;
    	}
    
    	return filepath;
    }
    

![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220704%2F1656941501_62c2ebbd65e6d66af456c.png%21small)

### 跟踪代码

发现SecurityUtil.pathFilter方法，先进入该方法-
-
发现有%号就进入（../这种特殊字符会被url编码成%），所以这是过滤特殊字符-
过滤后再判断是否有../，有的话返回null最后跳出方法，返回为null-
-
的话就输出 Bad boy.非法文件路径。![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220704%2F1656941511_62c2ebc7255c8651f687c.png%21small)![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220704%2F1656941570_62c2ec0237b6dd4760704.png%21small)-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220704%2F1656941534_62c2ebdebdec11f8c43ea.png%21small)-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220704%2F1656941582_62c2ec0ed313c86ffbb9a.png%21small)