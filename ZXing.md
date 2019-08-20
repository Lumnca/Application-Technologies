# :golf:ZXing  #

<b id="t"></b>

:arrow_double_down:[ZXing制作二维码与解码](#a1)

<b id="a1"></b>

### :bowling:ZXing制作二维码与解码 ###

:arrow_double_up: [返回目录](#t)

ZXing是一个java的二维码制作开源包，可以利用此包来制作二维码和解码二维码。zxing将生成图形编码的方式抽象成了一个类com.google.zxing.Writer, 在实现类中不仅仅生成二维码，还可以生成条形码等其他图形编码,如下方法

```java
BitMatrix encode(String contents, BarcodeFormat format, int width, int height) throws WriterException
BitMatrix encode(String contents, BarcodeFormat format, int width, int height, Map<EncodeHintType,?> hints) throws WriterException;
```

参数解释如下：

|参数|说明|
|:--|:---|
|String contents|编码的内容|
|BarcodeFormat format|编码的方式（二维码、条形码...）|
|int width|首选的宽度|
|int height|首选的高度|
|Map<EncodeHintType,?> hints|编码时的额外参数|

从上面可以看出，除了我们常规认为的编码需要内容之外，还有其他不少的信息，如编码的方式（这里只探讨二维码），二维码的首选宽高（首选的意思是：生成的图片的参考尺寸，如二维码是正方形，但给一个矩形，则会留白，条形码为矩形，设置一个正方形，则也会留白）。
下面详细讨论一下额外的参数，虽然不一定所有都用到，但是尽量讨论一些可能会用到的参数。编码额外的参数是以一个Map<EncodeHintType, ?>存在的，key为EncodeHintType枚举，那么可以看到所有的参数类型。


|参数|说明|
|:--|:---|
|ERROR_CORRECTION|容错率，指定容错等级，例如二维码中使用的ErrorCorrectionLevel, Aztec使用Integer|
|CHARACTER_SET|编码集|
|DATA_MATRIX_SHAPE|指定生成的数据矩阵的形状，类型为SymbolShapeHint|
|MARGIN|生成条码的时候使用，指定边距，单位像素，受格式的影响。类型Integer, 或String代表的数字类型|
|PDF417_COMPACT|指定是否使用PDF417紧凑模式（具体含义不懂）类型Boolean|
|PDF417_COMPACTION|指定PDF417的紧凑类型|
|PDF417_DIMENSIONS|指定PDF417的最大最小行列数|
|QR_VERSION|指定二维码版本，版本越高越复杂，反而不容易解析|

从上面的参数表格可以看出，适用于二维码的有：ERROR_CORRECTION, CHARACTER_SET, MARGIN, QR_VERSION.



|参数|使用说明|
|:--|:---|
|ERROR_CORRECTION|分为四个等级：L/M/Q/H, 等级越高，容错率越高，识别速度降低。例如一个角被损坏，容错率高的也许能够识别出来。通常为H|
|CHARACTER_SET|编码集，通常有中文，设置为 utf-8|
|MARGIN|默认为4, 实际效果并不是填写的值，一般默认值就行|
|QR_VERSION|通常不变，设置越高，反而不好用|


使用这个方法就可以生成BitMatrix数据，但是并不是二维码图片，要获得二维码，还需要转换为图片，如下方法:

```java
public static BufferedImage toBufferedImage(BitMatrix matrix, MatrixToImageConfig config)
```

其中参数BitMatrix为二维码的描述对象，为MatrixToImageConfig	二维码转换成BufferedImage的配置参数。

MatrixToImageConfig对象中只有两个域onColor和offColor, 文章开头提到二维码类似于二进制，这样的配置表示生成的BufferedImage用两种颜色来表示二维码上的开关。将BitMatrix转换成图片文件在官方提供的zxing-javase包中实际上有将BitMatrix转换成图片文件的方法，不过实际上是先将BitMatrix转换BufferedImage, 然后将其转换成图片文件。转换方法(javax.imageio.ImageIO)

```java
public static boolean write(RenderedImage im, String formatName, File output) throws IOException
```
参数说明：

```
RenderedImage im: BufferedImage实现了RenderedImage接口
String formatName:图片文件格式，通常使用 png
File output:图片文件
```

上面两步结合起来就直接将BitMatrix转换成文件了，下面是MatrixToImageWriter的方法(类型Path表示文件路径，可以使用File.toPath()方法得到)

```java
public static void writeToPath(BitMatrix matrix, String format, Path file, MatrixToImageConfig config) throws IOException
```



这里以Web应用为例：

可以下载包Zxing，我这里以Spring Boot工程添加：

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>core</artifactId>
            <version>3.2.1</version>
        </dependency>
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>javase</artifactId>
            <version>3.0.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.1.29</version>
        </dependency>
    </dependencies>
```

最后添加一个JSON解析依赖。接下来创建二维码创建模板类：

```java
package run;

import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.nio.file.FileSystems;
import java.nio.file.Path;
import java.util.HashMap;
import java.util.Map;
import javax.imageio.ImageIO;
import org.junit.Test;
import com.alibaba.fastjson.JSONObject;
import com.google.zxing.BarcodeFormat;
import com.google.zxing.Binarizer;
import com.google.zxing.BinaryBitmap;
import com.google.zxing.DecodeHintType;
import com.google.zxing.EncodeHintType;
import com.google.zxing.LuminanceSource;
import com.google.zxing.MultiFormatReader;
import com.google.zxing.MultiFormatWriter;
import com.google.zxing.NotFoundException;
import com.google.zxing.Result;
import com.google.zxing.WriterException;
import com.google.zxing.client.j2se.BufferedImageLuminanceSource;
import com.google.zxing.client.j2se.MatrixToImageWriter;
import com.google.zxing.common.BitMatrix;
import com.google.zxing.common.HybridBinarizer;

public class QrCodeTest {

    /**
     * 生成图像
     *
     * @throws WriterException
     * @throws IOException
     */
    @Test
    public void testEncode() throws WriterException, IOException {
        //装配路径
        String filePath = "D://";
        String fileName = "zxing.png";
        //以JSON字符串转入
        JSONObject json = new JSONObject();
        json.put("name","Lumnca");
        json.put("infor", "GOOD");
        String content = json.toJSONString();// 内容
        int width = 200; // 图像宽度
        int height = 200; // 图像高度
        String format = "png";// 图像类型
        Map<EncodeHintType, Object> hints = new HashMap<>();
        hints.put(EncodeHintType.CHARACTER_SET, "UTF-8");
        BitMatrix bitMatrix = new MultiFormatWriter().encode(content, BarcodeFormat.QR_CODE, width, height, hints);// 生成矩阵
        //本地路径
        Path path = FileSystems.getDefault().getPath(filePath, fileName);
        /*
        Web路径
        File file = new File(this.getClass().getResource("/").getPath()+"/pr.png");
        Path path =   file.toPath();
        */
        MatrixToImageWriter.writeToPath(bitMatrix, format, path);// 输出图像
        System.out.println("输出成功.");
    }

    /**
     * 解析图像
     */
    @Test
    public void testDecode() {
        String filePath = "D://zxing.png"; //路径
        BufferedImage image;
        try {
            image = ImageIO.read(new File(filePath));
            LuminanceSource source = new BufferedImageLuminanceSource(image);
            Binarizer binarizer = new HybridBinarizer(source);
            BinaryBitmap binaryBitmap = new BinaryBitmap(binarizer);
            Map<DecodeHintType, Object> hints = new HashMap<>();
            hints.put(DecodeHintType.CHARACTER_SET, "UTF-8");
            Result result = new MultiFormatReader().decode(binaryBitmap, hints);// 对图像进行解码，获取文本数据
            JSONObject content = JSONObject.parseObject(result.getText());
            System.out.println("图片中内容：  ");
            System.out.println("author： " + content.getString("name"));
            System.out.println("zxing：  " + content.getString("infor"));
            System.out.println("图片中格式：  ");
            System.out.println("encode： " + result.getBarcodeFormat());
            System.out.println(result);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (NotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

后面直接调用就行：

```java
@RestController

public class index {
    @GetMapping("getQr")
    public String getQr()throws Exception{
        QrCodeTest Qr = new QrCodeTest();
        Qr.testEncode();
        return "生成成功！";
    }
    @GetMapping("Qr")
    public String Qr(){
        QrCodeTest Qr = new QrCodeTest();
        Qr.testDecode();
        return "解析完毕";
    }
}
```
