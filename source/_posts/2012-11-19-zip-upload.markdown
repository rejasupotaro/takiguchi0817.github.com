---
layout: post
title: "Androidで複数画像をzipにしてmultipart/form-dataで送ってRubyで受け取る"
date: 2012-11-19 01:08
comments: false
categories: Android Ruby
---

## Androidで送る

事情があって一回のリクエストで複数画像を送りたかった。  

### UriからStringの変換

画像のパスが必要になるので、ContentResolverに問い合わせてパスもらう。  

{% codeblock lang:java %}
private String getPath(Uri uri) {
    ContentResolver contentResolver = mContext.getContentResolver();
    String[] columns = { MediaStore.Images.Media.DATA };
    Cursor cursor = null;
    String path = null;
    try {
        cursor = contentResolver.query(uri, columns, null, null, null);
        cursor.moveToFirst();
        path = cursor.getString(0);
    } finally {
        if (cursor != null) cursor.close();
    }
    return path;
}
{% endcodeblock %}

### Closableを閉じる

finallyで閉じてもいいけどtryが入れ子になるのなんか嫌なのでメソッド作る。  

{% codeblock lang:java %}
public static void close(Closeable closeable) {
        if (closeable == null) return;
        try {
            closeable.close();
        } catch (IOException e) {
            Log.e(TAG, "An error occurred in CloseableUtils.close()", e);
        }
    }
}
{% endcodeblock %}

#### before:  

{% codeblock lang:java %}
} finally {
    if (hogeStream != null) {
        try {
            hogeStream.close();
        } catch (IOException e) {
            Log.e(TAG, "omg!!", e);
        }
    }
}
{% endcodeblock %}

#### after:  

{% codeblock lang:java %}
} finally {
    CloseableUtils.close(hogeStream);
}
{% endcodeblock %}

すっきりした。  

### zipにする

HTTPに画像を配列にしてポストする、みたいなのなかったように思える。なのでzipにした。  

{% codeblock lang:java %}
private File toZip(String outputFilePath, List<Uri> inputFileUriList) {
    File oldFile = new File(outputFilePath);
    if (oldFile.isFile()) oldFile.delete();

    ZipOutputStream zipOutputStream = null;
    BufferedInputStream bufferedInputStream = null;
    try {
        zipOutputStream = new ZipOutputStream(
                new BufferedOutputStream(new FileOutputStream(outputFilePath)));

        byte[] buffer = new byte[BUFFER_SIZE];
        for (int i = 0; i < inputFileUriList.size(); i++) {
            String filePath = getPath(inputFileUriList.get(i));
            bufferedInputStream = new BufferedInputStream(
                    new FileInputStream(new File(filePath)), BUFFER_SIZE);
            final ZipEntry entry = new ZipEntry(i +  ".jpg");
            zipOutputStream.putNextEntry(entry);
            int len = 0;
            while ((len = bufferedInputStream.read(buffer, 0, BUFFER_SIZE)) != -1) {
                zipOutputStream.write(buffer, 0, len);
            }
            zipOutputStream.closeEntry();
        }
    } catch (FileNotFoundException e) {
        Log.e(TAG, e.getMessage());
    } catch (IOException e) {
        Log.e(TAG, e.getMessage());
    } finally {
        CloseableUtils.close(zipOutputStream);
        CloseableUtils.close(bufferedInputStream);
    }

    File result = new File(outputFilePath);
    return result;
}
{% endcodeblock %}

これでout.zipが作られて解凍すると1.jpg、2.jpg ... みたいにファイルできる。  
outFilePathはどこでもいいんだけど、  

{% codeblock lang:java %}
context.getExternalCacheDir().getPath() + "/out.zip"
{% endcodeblock %}

とかしてSDカードのキャッシュに保存した。  

### multipart/form-dataで送る

ライブラリとして  
* Apache Mime4J  
* httpmime  
追加した。  

{% codeblock lang:java %}
final HttpPost httpPost = new HttpPost(url);

final MultipartEntity reqEntity =
        new MultipartEntity(HttpMultipartMode.BROWSER_COMPATIBLE);
reqEntity.addPart(titleNameValuePair.getName(),
        new StringBody(titleNameValuePair.getValue(), DEFAULT_CHARSET));
reqEntity.addPart(delayNameValuePair.getName(),
        new StringBody(delayNameValuePair.getValue(), DEFAULT_CHARSET));
final File file = new File(fileNameValuePair.getValue());
reqEntity.addPart(fileNameValuePair.getName(),
        new FileBody(file, CONTENTTYPE_BINARY));

httpPost.setEntity(reqEntity);
{% endcodeblock %}

こんな感じでエンティティ〜作ってHttpClientでポストする。  

## Rubyで受け取る

Sinatraだったら  

{% codeblock lang:ruby %}
require 'rubygems'
require 'sinatra'

set :public, File.dirname(__FILE__) + '/public'

post '/' do
  zipfile = params['zipfile']
  File.binwrite('public/' + zipfile[:filename], zipfile[:tempfile])
end
{% endcodeblock %}

とかするといける。  

## このプログラムはフォンデュしながら書きました

![チョコフォンデュ](http://dl.dropbox.com/u/54255753/blog/201211/fondue.gif)
