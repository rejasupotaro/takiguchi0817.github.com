---
layout: post
title: "Androidの画像の暗号化"
date: 2012-10-06 17:05
comments: false
categories: Android Encryption
---

## SDカードに保存する画像を暗号化したい


### Androidの暗号化手法

Androidで使用出来る暗号化手法一覧：Security.getAlgorithms("Cipher")
    PBEWITHSHAAND192BITAES-CBC-BC
    PBEWITHSHAAND40BITRC4
    PBEWITHSHAAND128BITRC2-CBC
    PBEWITHSHA256AND256BITAES-CBC-BC
    PBEWITHSHAAND128BITAES-CBC-BC
    PBEWITHSHAAND40BITRC2-CBC
    PBEWITHSHA1ANDDES
    AESWRAP
    DES
    AES
    DESEDEWRAP
    PBEWITHSHAAND256BITAES-CBC-BC
    PBEWITHSHA256AND192BITAES-CBC-BC
    PBEWITHMD5AND128BITAES-CBC-OPENSSL
    PBEWITHMD5AND256BITAES-CBC-OPENSSL
    BLOWFISH
    PBEWITHSHAAND2-KEYTRIPLEDES-CBC
    PBEWITHSHA1ANDRC2
    PBEWITHMD5AND192BITAES-CBC-OPENSSL
    ARC4
    PBEWITHMD5ANDDES
    RSA
    PBEWITHSHA256AND128BITAES-CBC-BC
    PBEWITHSHAAND3-KEYTRIPLEDES-CBC
    DESEDE
    PBEWITHSHAANDTWOFISH-CBC
    PBEWITHSHAAND128BITRC4
    PBEWITHMD5ANDRC2
色々あるが、今回はAESを検証した。

### コード

定数
    private static final byte[] SALT = new byte[] {
        -47, 66, 32, -127, -102, -51, 79, -69, 57, 85, -91, -42, 74, -116 };
    private static final byte[] INITAIL_VECTOR = {
        16, 74, 71, -80, 32, 101, -47, 72, 117, -14, 0, -29, 70, 65, -12, 74 };
    private static final String CIPHER_ALGORITM = "AES/CBC/PKCS5Padding";

鍵の生成
    private static SecretKey generateKey()
            throws  NoSuchAlgorithmException, InvalidKeySpecException {  
    
        // ここでは"password"を与えているが実際にやってはいけない
        char[] password = "password".toCharArray();
    
        // ソルト、イテレーションカウント1024回、鍵の長さ256ビットを指定
        KeySpec keySpec = new PBEKeySpec(password, SALT, 1024, 256);  
        SecretKeyFactory factory = SecretKeyFactory.getInstance("PBEWITHSHAAND256BITAES-CBC-BC");  
        SecretKey secretKey = factory.generateSecret(keySpec);  
    
        return secretKey;  
    }

暗号化
    public static byte[] encrypt(Bitmap bitmap)
            throws
            NoSuchAlgorithmException, InvalidKeySpecException, NameNotFoundException,
            InvalidKeyException, NoSuchPaddingException, InvalidAlgorithmParameterException,
            IllegalBlockSizeException, BadPaddingException {

        SecretKey secretKey = generateKey();

        // 暗号アルゴリズムにAESを指定
        Cipher cipher = Cipher.getInstance(CIPHER_ALGORITM);

        // 暗号化モードに設定し、キーを指定
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, new IvParameterSpec(INITAIL_VECTOR));

        // 暗号化の実行
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        bitmap.compress(CompressFormat.JPEG, 100, bos);
        return cipher.doFinal(bos.toByteArray());
    }

復号
    public static Bitmap decrypt(byte[] crypted)
            throws
            NoSuchAlgorithmException, InvalidKeySpecException, NoSuchPaddingException,
            InvalidKeyException, InvalidAlgorithmParameterException,
            IllegalBlockSizeException, BadPaddingException {

        SecretKey secretKey = generateKey();

        // 暗号アルゴリズムにAESを指定
        Cipher cipher = Cipher.getInstance(CIPHER_ALGORITM);

        // 暗号化モードに設定し、キーを指定
        cipher.init(Cipher.DECRYPT_MODE, secretKey, new IvParameterSpec(INITAIL_VECTOR));

        // 復号の実行
        byte[] decrypted = cipher.doFinal(crypted);
        BitmapFactory.Options options = new BitmapFactory.Options();
        return BitmapFactory.decodeByteArray(decrypted, 0, decrypted.length, options);
    }

### 検証
500 * 500 の画像10枚の暗号化と復号にかかった時間を測定…しようと思って実行したらアプリが応答しなかった。  

traceviewで画像1枚の処理時間を調べたら、全体で6800msec掛かっていた。encrypt(generateKey込み)が80%、decryptが20%で、generateKeyにかなりの時間がかかっていた。  
なのでsecretKeyを外で生成して暗号化・復号に掛かる時間を計測してみたら2800msecになった。  

### 感想
画像を暗号化するなら、ローカルに置くよりサーバに置いた方が速いし楽な気がする。  
絶対に見られてはならない機密情報ってほどでもないが、あんまり見られたくないという画像は、[ヘッダ](http://www.kk.iij4u.or.jp/~kondo/bmp/)をいじるのが良いのだろうか。  

画像に関してはあれだが、アクセストークンくらいなら速いししばらくメモリに乗ってそうだし、良さそう。  
注意する点として、SharedPreferenceにはStringしか保存できないので、暗号化と復号の処理の間にBase64化処理が必要になる。  
しかもAndroidのBase64クラスが使えるのは2.2以降なので、そういうのをうまくやる必要がある。
