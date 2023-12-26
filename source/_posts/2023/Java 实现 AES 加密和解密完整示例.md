---
title: Java 实现 AES 加密和解密完整示例
author: ehzyil
tags:
  - AES
  - Java
categories:
  - 技术
date: 2023-12-26


---

## Java 实现 AES 加密和解密完整示例

> 遇到需求需要给已AES加密的用户姓名解密，并给出了python代码，网上找了个很全的Java 实现 AES 加密和解密完整示例，记录以下方便以后查阅。
>
> 转载[Java 实现 AES 加密和解密完整示例](https://blog.csdn.net/u011374856/article/details/130635405)
>
> [其他详细资料](https://www.cnblogs.com/better-farther-world2099/p/13293291.html)



### 1、简介
AES，全称为 Advanced Encryption Standard，是一种分组密码算法，用于保护敏感数据的传输和存储。AES 分为 128 位和 256 位两种密钥长度，可以对数据进行加密和解密，保证数据的安全性和完整性。AES 主要应用于电子商务、移动支付、网络安全等领域，被广泛运用于现代社会的各个方面。AES 算法被设计为高度安全，可以在理论上保证其分组密码的安全性。然而，由于其复杂性和密钥长度，AES 算法的实现和应用也具有一定的技术难度。因此，在应用 AES算法时，需要注意加强密钥管理和安全性保障。

这个标准用来替代原先的 DES（Data Encryption Standard），已经被多方分析且广为全世界所使用。

AES 算法具有很多优点，例如快速、安全、可靠等。它可以加密大量数据，而不会因为加密过程中的数据量过大而变得缓慢。此外，AES 算法还支持块大小的自动调整，可以处理不同大小的数据块。

### 2、AES 加密模式
**2.1、加密方式**
ECB（Electronic Codebook）模式：这种模式是将整个明文分成若干段相同的小段，然后对每一小段进行加密。加密时，使用一个密钥，将明文中的每个字符与密钥中对应位置的字符进行异或运算，得到密文。

CBC（Cipher Block Chaining）模式：这种模式是先将明文切分成若干小段，然后每一小段与初始块或者上一段的密文段进行异或运算后，再与密钥进行加密。加密时，使用一个密钥和一个初始化向量（IV），初始化向量是一个16字节的向量，包含了加密算法所需的所有信息。

CFB（Cipher Feedback）模式：这种模式是一种较为复杂的加密模式，它结合了CBC和CTR两种模式的优点。在CFB模式中，加密过程中使用一个密钥和一个随机生成的初始化向量（IV），然后对明文进行加密。在加密完成后，通过对明文进行非对称加密来生成密文的向量。随后，通过对密文进行反向操作，将密文的向量与明文的向量进行异或运算，得到解密所需的密钥。

需要注意的是，ECB、CBC、CFB等模式都是对称加密算法，加密和解密使用相同的密钥。在使用这些算法时，需要注意保护密钥的安全，避免被恶意获取。

**2.2、安全性**
ECB 不够安全，只适合于短数据的加密，而 CBC 和 CFB 相较于 ECB 更加安全，因为前一个密文块会影响当前明文块，使攻击者难以预测密文的结构。

**2.3、速度**
ECB 是最简单的加密方式，速度最快，但由于安全性差不建议使用，CBC 因为每个明文块都要与前一个密文块进行异或操作，比 ECB 要慢一些，CFB 因为需要反复加密和解密，速度可能会更慢。

总的来说，选择 AES 的算法模式需要根据加密需要的安全性和速度来进行选择，通常推荐使用CBC 或 CFB 模式，而不是 ECB 模式。

### 3、Java 实现完整示例
在 Java 中，可以使用 javax.crypto 包中的 Cipher 类来实现 AES 加密和解密。完整代码如下：

```java

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.Base64;
import java.util.Random;

/**
 * <h1>AES 加密和解密示例代码</h1>
 * Created by woniu
 * */
public class AESExample {

    /** 加密模式之 ECB，算法/模式/补码方式 */
    private static final String AES_ECB = "AES/ECB/PKCS5Padding";

    /** 加密模式之 CBC，算法/模式/补码方式 */
    private static final String AES_CBC = "AES/CBC/PKCS5Padding";

    /** 加密模式之 CFB，算法/模式/补码方式 */
    private static final String AES_CFB = "AES/CFB/PKCS5Padding";

    /** AES 中的 IV 必须是 16 字节（128位）长 */
    private static final Integer IV_LENGTH = 16;

    /***
     * <h2>空校验</h2>
     * @param str 需要判断的值
     */
    public static boolean isEmpty(Object str) {
        return null == str || "".equals(str);
    }

    /***
     * <h2>String 转 byte</h2>
     * @param str 需要转换的字符串
     */
    public static byte[] getBytes(String str){
        if (isEmpty(str)) {
            return null;
        }

        try {
            return str.getBytes(StandardCharsets.UTF_8);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }


    /***
     * <h2>初始化向量（IV），它是一个随机生成的字节数组，用于增加加密和解密的安全性</h2>
     */
    public static String getIV(){
//        String str = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
//        Random random = new Random();
//        StringBuffer sb = new StringBuffer();
//        for(int i = 0 ; i < IV_LENGTH ; i++){
//            int number = random.nextInt(str.length());
//            sb.append(str.charAt(number));
//        }
//        return sb.toString();
        return "1234567876543210";
    }

    /***
     * <h2>获取一个 AES 密钥规范</h2>
     */
    public static SecretKeySpec getSecretKeySpec(String key){
        SecretKeySpec secretKeySpec = new SecretKeySpec(getBytes(key), "AES");
        return secretKeySpec;
    }

    /**
     * <h2>加密 - 模式 ECB</h2>
     * @param text 需要加密的文本内容
     * @param key 加密的密钥 key
     * */
    public static String encrypt(String text, String key){
        if (isEmpty(text) || isEmpty(key)) {
            return null;
        }

        try {
            // 创建AES加密器
            Cipher cipher = Cipher.getInstance(AES_ECB);

            SecretKeySpec secretKeySpec = getSecretKeySpec(key);

            cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);

            // 加密字节数组
            byte[] encryptedBytes = cipher.doFinal(getBytes(text));

            // 将密文转换为 Base64 编码字符串
            return Base64.getEncoder().encodeToString(encryptedBytes);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * <h2>解密 - 模式 ECB</h2>
     * @param text 需要解密的文本内容
     * @param key 解密的密钥 key
     * */
    public static String decrypt(String text, String key){
        if (isEmpty(text) || isEmpty(key)) {
            return null;
        }

        // 将密文转换为16字节的字节数组
        byte[] textBytes = Base64.getDecoder().decode(text);

        try {
            // 创建AES加密器
            Cipher cipher = Cipher.getInstance(AES_ECB);

            SecretKeySpec secretKeySpec = getSecretKeySpec(key);

            cipher.init(Cipher.DECRYPT_MODE, secretKeySpec);

            // 解密字节数组
            byte[] decryptedBytes = cipher.doFinal(textBytes);

            // 将明文转换为字符串
            return new String(decryptedBytes, StandardCharsets.UTF_8);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * <h2>加密 - 自定义加密模式</h2>
     * @param text 需要加密的文本内容
     * @param key 加密的密钥 key
     * @param iv 初始化向量
     * @param mode 加密模式
     * */
    public static String encrypt(String text, String key, String iv, String mode){
        if (isEmpty(text) || isEmpty(key) || isEmpty(iv)) {
            return null;
        }

        try {
            // 创建AES加密器
            Cipher cipher = Cipher.getInstance(mode);

            SecretKeySpec secretKeySpec = getSecretKeySpec(key);

            cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec, new IvParameterSpec(getBytes(iv)));

            // 加密字节数组
            byte[] encryptedBytes = cipher.doFinal(getBytes(text));

            // 将密文转换为 Base64 编码字符串
            return Base64.getEncoder().encodeToString(encryptedBytes);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * <h2>解密 - 自定义加密模式</h2>
     * @param text 需要解密的文本内容
     * @param key 解密的密钥 key
     * @param iv 初始化向量
     * @param mode 加密模式
     * */
    public static String decrypt(String text, String key, String iv, String mode){
        if (isEmpty(text) || isEmpty(key) || isEmpty(iv)) {
            return null;
        }

        // 将密文转换为16字节的字节数组
        byte[] textBytes = Base64.getDecoder().decode(text);

        try {
            // 创建AES加密器
            Cipher cipher = Cipher.getInstance(mode);

            SecretKeySpec secretKeySpec = getSecretKeySpec(key);

            cipher.init(Cipher.DECRYPT_MODE, secretKeySpec, new IvParameterSpec(getBytes(iv)));

            // 解密字节数组
            byte[] decryptedBytes = cipher.doFinal(textBytes);

            // 将明文转换为字符串
            return new String(decryptedBytes, StandardCharsets.UTF_8);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }


    public static void main(String[] args) {
        String text = "411025200102175014";
        String key  = "rHx3pZwRTAb33psX"; // 16字节的密钥
        String iv  = getIV();



        String encryptTextEBC = encrypt(text, key);
        System.out.println("EBC 加密后内容：" + encryptTextEBC);
        System.out.println("EBC 解密后内容：" + decrypt(encryptTextEBC, key));
        System.out.println();

        String encryptTextCBC = encrypt(text, key, iv, AES_CBC);
        System.out.println("CBC 加密IV：" + iv);
        System.out.println("CBC 加密后内容：" + encryptTextCBC);
        System.out.println("CBC 解密后内容：" + decrypt(encryptTextCBC, key, iv, AES_CBC));
        System.out.println();

        String encryptTextCFB = encrypt(text, key, iv, AES_CFB);
        System.out.println("CFB 加密IV：" + iv);
        System.out.println("CFB 加密后内容：" + encryptTextCFB);
        System.out.println("CFB 解密后内容：" + decrypt(encryptTextCFB, key, iv, AES_CFB));
    }
}

    public static String decryptAES(String text) {
        if (StringUtils.isEmpty(text)) return "";
        text = text.substring(4);
        System.out.println(text);
        String mode = "AES/CBC/PKCS5Padding";
        String iv = "1234567876543210";
        String key = "rHx3pZwRTAb33psX"; // 16字节的密钥
        // 将密文转换为16字节的字节数组
        byte[] textBytes = Base64.getDecoder().decode(text);

        try {
            // 创建AES加密器
            Cipher cipher = Cipher.getInstance(mode);

            SecretKeySpec secretKeySpec = getSecretKeySpec(key);

            cipher.init(Cipher.DECRYPT_MODE, secretKeySpec, new IvParameterSpec(getBytes(iv)));

            // 解密字节数组
            byte[] decryptedBytes = cipher.doFinal(textBytes);

            // 将明文转换为字符串
            return new String(decryptedBytes, StandardCharsets.UTF_8);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

    }
```





<br>

需求中给出的python代码

AesEncryption.py

```python
import base64  # 导入base64模块
from Crypto.Cipher import AES  # 从Crypto模块中导入AES加密算法

import StringUtil  # 导入一个自定义的StringUtil模块


class AesEncryption:
    __AES_IV = '1234567876543210'  # 初始化一个AES IV向量

    def __init__(self, password: str = 'rHx3pZwRTAb33psX', prefix: str = 'AES:', encoding_protocol: str = 'ASCII'):
        self.__password = password  # 初始化密码
        self.__encoding_protocol = encoding_protocol  # 初始化编码协议
        self.__prefix = prefix  # 初始化前缀

    # AES加密方法
    def aes_encrypt(self, text: str):
        block_size = 16  # 定义块大小为16
        # 数据进行 PKCS5Padding 的填充
        pad = lambda s: (s + (block_size - len(s) % block_size) * chr(block_size - len(s) % block_size))  # 定义填充函数
        data = pad(text)  # 对文本进行填充
        # 创建加密对象
        cipher = AES.new(self.__password.encode(), AES.MODE_CBC, self.__AES_IV.encode())  # 创建AES加密对象
        # 得到加密后的字节码
        encrypted_text = cipher.encrypt(bytes(data, self.__encoding_protocol))  # 进行加密
        # base64 编码
        encode_text = base64.b64encode(encrypted_text).decode(self.__encoding_protocol)  # 进行base64编码
        if StringUtil.is_str_not_null(self.__prefix):  # 判断前缀是否不为空
            encode_text = self.__prefix + encode_text  # 如果前缀不为空，则添加前缀
        return encode_text  # 返回加密后的文本

    # AES解密方法
    def aes_decrypt(self, text: str):
        # 去掉 PKCS5Padding 的填充
        un_pad = lambda s: s[:-ord(s[len(s) - 1:])]  # 定义去除填充的函数
        # 创建加密对象
        cipher = AES.new(self.__password.encode(), AES.MODE_CBC, self.__AES_IV.encode())  # 创建AES解密对象
        # base64 解码
        text = StringUtil.split_by_first_str(text, self.__prefix)  # 通过前缀分割文本

        decode_text = base64.b64decode(str(text).encode())  # 进行base64解码
        decrypt_text = un_pad(cipher.decrypt(decode_text)).decode(self.__encoding_protocol)  # 进行解密
        return decrypt_text  # 返回解密后的文本

    # 类方法：AES加密
    @classmethod
    def aes_encrypt_cls(cls, text: str, password: str = 'rHx3pZwRTAb33psX', prefix: str = 'AES:', encoding_protocol: str = 'ASCII'):
        aes_en = AesEncryption(password=password, prefix=prefix, encoding_protocol=encoding_protocol)  # 创建AesEncryption实例
        return aes_en.aes_encrypt(text)  # 调用实例方法进行加密

    # 类方法：AES解密
    @classmethod
    def aes_decrypt_cls(cls,  text: str, password: str = 'rHx3pZwRTAb33psX', prefix: str = 'AES:', encoding_protocol: str = 'ASCII'):
        aes_de = AesEncryption(password=password, prefix=prefix, encoding_protocol=encoding_protocol)  # 创建AesEncryption实例
        return aes_de.aes_decrypt(text)  # 调用实例方法进行解密
    

    # 要加密的文本
text_to_encrypt = "411025200102175014"

# 使用类方法进行加密
encrypted_text = AesEncryption.aes_encrypt_cls(text_to_encrypt)

# 输出加密后的文本
print("Encrypted Text:", encrypted_text)

# 使用类方法进行解密
decrypted_text = AesEncryption.aes_decrypt_cls(encrypted_text)

# 输出解密后的文本
print("Decrypted Text:", decrypted_text)


#StringUtil.py
def is_str_not_null(text):
    if text is None:
        return False
    else:
        if type(text) == str:
            if text.strip() == '':
                return False
            else:
                return True
        else:
            raise Exception('text需为字符串')


def split_by_first_str(text, prefix):
    if is_str_not_null(text) and is_str_not_null(prefix):
        sub_arr = text.split(prefix, 1)
        if len(sub_arr) > 1:
            return sub_arr[1]
        else:
            return text
    else:
        return text

```

