# :golf:ZXing  #

<b id="t"></b>

:arrow_double_down:[ZXing制作二维码与解码](#a1)

<b id="a1"></b>

### :bowling:ZXing制作二维码与解码 ###

:arrow_double_up: [返回目录](#t)

ZXing是一个java的二维码制作开源包，可以利用此包来制作二维码和解码二维码。

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
        json.put("name","高巨");
        json.put("infor", "你是真的铁憨憨");
        String content = json.toJSONString();// 内容
        int width = 200; // 图像宽度
        int height = 200; // 图像高度
        String format = "png";// 图像类型
        Map<EncodeHintType, Object> hints = new HashMap<>();
        hints.put(EncodeHintType.CHARACTER_SET, "UTF-8");
        BitMatrix bitMatrix = new MultiFormatWriter().encode(content,
                BarcodeFormat.QR_CODE, width, height, hints);// 生成矩阵
        Path path = FileSystems.getDefault().getPath(filePath, fileName);
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
