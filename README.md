# article.xml文件结构
```
<?xml version="1.0" encoding="UTF-8"?>
<articleStore>
    <article articleId="1">
        <title>如何提升 Java 技能</title>
        <author>geshan</author>
        <content>关注公众号：格姗知识圈，了解更多技术文</content>
    </article>

    <article articleId="2">
        <title>如何高效使用软件工具</title>
        <author>小格子</author>
        <content>关注公众号：小格子实用技巧，了解更多实用工具</content>
    </article>
</articleStore>
```

# dom4j-util 工具类
使用dom4j读取XML文件内容和创建XML文件 

```
package geshanzsq.utils;

import com.alibaba.fastjson.JSONObject;
import geshanzsq.entity.Article;
import org.dom4j.*;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.SAXReader;
import org.dom4j.io.XMLWriter;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Date;
import java.util.Iterator;
import java.util.List;

/**
 * dom4j 工具类
 * @author geshanzsq
 * @date 2020/3/3
 */
public class Dom4jUtils {

    public static void main(String[] args) {

        //读取当前项目目录resources下的article.xml文件
        String filePath = "E:\\IntelliJ IDEA\\MyWorkSpace\\dom4j-util\\src\\main\\resources\\article.xml";
        //读取XML文件内容
        List<Article> articleList = getArticleList(filePath);
        System.out.println(JSONObject.toJSONString(articleList));

        //创建XML文件
        //文件保存路径
        String xmlPath = "E:\\IntelliJ IDEA\\MyWorkSpace\\dom4j-util\\src\\main\\resources";
        //文件保存名
        String fileName = new Date().getTime() + "_article.xml";
        //创建XML文件
        createArticleXml(xmlPath,fileName,articleList);

    }


    /**
     * 读取XML文件内容
     * @param filePath
     */
    public static List<Article> getArticleList(String filePath) {

        //返回数据集合
        List<Article> articleList = new ArrayList<Article>();

        //获取 XML 文件流
        File articleFile = new File(filePath);

        try {
            //读取XML文件,获得document对象
            SAXReader saxReader = new SAXReader();
            Document document = saxReader.read(articleFile);

            //获取根目录，即对应此XML的articleStore
            Element articleStoreElement = document.getRootElement();
            Iterator articleStoreIt = articleStoreElement.elementIterator();

            //遍历根目录下的article
            while (articleStoreIt.hasNext()) {

                Article article = new Article();
                Element articleElement = (Element)articleStoreIt.next();

                //遍历article的属性
                List<Attribute> articleAttributes = articleElement.attributes();
                for (Attribute attribute: articleAttributes) {
                    //属性名称
                    String attributeName = attribute.getName();
                    if ("articleId".equals(attributeName)) {
                        //文章id
                        String articleId = attribute.getValue();
                        article.setArticleId(Long.parseLong(articleId));
                    }
                }

                //遍历article下的标签
                Iterator iterator = articleElement.elementIterator();
                while (iterator.hasNext()) {
                    Element articleChild = (Element) iterator.next();
                    //获取标签名
                    String childName = articleChild.getName();

                    if ("title".equals(childName)) {
                        //标题
                        String title = articleChild.getStringValue();
                        article.setTitle(title);
                    } else if ("author".equals(childName)) {
                        //作者
                        String author = articleChild.getStringValue();
                        article.setAuthor(author);
                    } else if ("content".equals(childName)) {
                        //内容
                        String content = articleChild.getStringValue();
                        article.setContent(content);
                    }
                }
                articleList.add(article);
            }
        } catch (DocumentException e) {
            e.printStackTrace();
        }
        return articleList;
    }

    /**
     * 创建XML文件
     * @param filePath 文件存储路径
     * @param fileName 文件存储名称
     * @param articles xml相关内容
     */
    public static void createArticleXml(String filePath,String fileName,List<Article> articles) {
        //创建文档模型
        Document document = DocumentHelper.createDocument();

        //定义一个根节点articleStore
        Element articleStoreElement = document.addElement("articleStore");

        //循环设置XML内容
        for (Article article: articles) {
            //跟节点下添加子节点article
            Element articleElement = articleStoreElement.addElement("article");

            //article节点添加属性
            Long articleId = article.getArticleId();
            articleElement.addAttribute("articleId", String.valueOf(articleId));

            //article节点添加子节点并设置节点值
            //设置子节点title
            Element titleElement = articleElement.addElement("title");
            titleElement.setText(article.getTitle());
            //设置子节点author
            Element authorElement = articleElement.addElement("author");
            authorElement.addText(article.getAuthor());
            //设置子节点content
            Element contentElement = articleElement.addElement("content");
            contentElement.addText(article.getContent());
        }

        //创建XML格式
        OutputFormat outputFormat = new OutputFormat();
        //编码格式
        outputFormat.setEncoding("UTF-8");
        //设置是否换行
        outputFormat.setNewlines(true);
        //设置是否缩进
        outputFormat.setIndent(true);
        //以4个空格方式实现缩进
        outputFormat.setIndent("    ");

        //创建文件流并写入数据
        File file = new File(filePath,fileName);
        try {
            FileWriter fileWriter = new FileWriter(file);
            XMLWriter xmlWriter = new XMLWriter(fileWriter,outputFormat);
            xmlWriter.write(document);
            xmlWriter.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}

```
