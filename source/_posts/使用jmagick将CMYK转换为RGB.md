title: "使用jmagick将CMYK转换为RGB"
date: 2013-05-23 00:27
category: [java]
tags: [jmagick]
---

最近发现公司图片库中有些打水印的图片水印色彩和其他的不一样，后来发现是设计师上传的图片的色彩值是CMYK的，想要统一成RGB格式的。
之前使用的是jmagick进行的图片压缩和水印，看了一下jmagick的官方介绍，是有提供转换图片色彩格式的方法。记录一下，给碰到类似问题的同学提供参考。
 
jmagick的ColorspaceType里面是色彩格式对应的常量。

```java
public static final int GRAYColorspace = 2;
public static final int TransparentColorspace = 3;
public static final int OHTAColorspace = 4;
public static final int XYZColorspace = 5;
public static final int YCbCrColorspace = 6;
public static final int YCCColorspace = 7;
public static final int YIQColorspace = 8;
public static final int YPbPrColorspace = 9;
public static final int YUVColorspace = 10;
public static final int CMYKColorspace = 11;
public static final int sRGBColorspace = 12;
```

将图片创建成MagickImage对象
 
ImageInfo imageInfo = new ImageInfo(filePath);
MagickImage fromImage = new MagickImage(imageInfo);
 
然后通过fromImage.getColorspace()可以拿到色彩格式
 
完整代码
```java
/**
	 * jmagick 将所有图片色彩统一为RGB
	 * @param filePath 原图路径
	 * @param toFilePath 转换后的图片路径
	 * @return
	 * @throws Exception
	 */
    public static InputStream convert2RGB(String filePath, String toFilePath) throws Exception{
        InputStream stream = null;
        ImageInfo imageInfo = new ImageInfo(filePath);
        MagickImage fromImage = new MagickImage(imageInfo);
        if(fromImage.getColorspace() != ColorspaceType.RGBColorspace){
            //因为是将所有其他格式转换为RGB格式，需要将当前文件的色彩格式传入
            fromImage.transformRgbImage(fromImage.getColorspace());
            fromImage.setFileName(toFilePath);
            fromImage.writeImage(imageInfo);
            stream = new FileInputStream(toFilePath);
            return stream;
        }
        return new FileInputStream(new File(filePath));
    }
```