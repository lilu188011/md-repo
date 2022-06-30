## GSON多层嵌套json解析

### 一、 简介

```
Google Gson是一个简单的基于Java的库，用于将Java对象序列化为JSON，反之亦然。 它是由
Google开发的一个开源库。
以下几点说明为什么应该使用这个库 -
OGNL表达式是Object-Graph Navigation Language的缩写，是一种功能强大的表达式语言，通过
简单一致的表达式语法，可以存取对象的任意属性，调用对象的方法，遍历整个对象的结构图，实现
字段类型转换。
```

### 二、 POM导入

```
		<dependency>
            <groupId>ognl</groupId>
            <artifactId>ognl</artifactId>
            <version>3.1.1</version>
        </dependency>
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.9.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
        </dependency>
```

### 三、工具类

```java
import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;
import ognl.Ognl;
import ognl.OgnlContext;

import java.util.Map;

public class JsonUtils {
    /**
     * 将指定JSON转为Map对象，Key固定为String，对应JSONkey
     * Value分情况：
     * 1. Value是字符串，自动转为字符串,例如:{"a"："b"}
     * 2. Value是其他JSON对象，自动转为Map，例如：{"a":{"b":"2"}}}
     * 3. Value是数组，自动转为List<Map>，例如：{"a":[{"b":"2"},{"c":"3"}]}
     * @param json 输入的JSON对象
     * @return 动态的Map集合
     */
    public static Map<String,Object> transferToMap(String json) {
        Gson gson = new Gson();
        Map<String, Object> map = gson.fromJson(json,
                new TypeToken<Map<String, Object>>() {
                }.getType());
        return map;
    }

    /**
     * 简化方法
     * @param json 原始的JSON数据
     * @param path OGNL规则表达式
     * @param clazz Value对应的目标类
     * @return clazz对应数据
     */
    public static <T> T getValue(String json, String path, Class<T> clazz) {
        try {
            Map map = transferToMap(json);
            OgnlContext context = new OgnlContext();
            context.setRoot(map);
            T value = (T) Ognl.getValue(path, context, context.getRoot());
            return value;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static <T> T getValueFromMap(Map map, String path, Class<T> clazz) {
        try {
            OgnlContext context = new OgnlContext();
            context.setRoot(map);
            T value = (T) Ognl.getValue(path, context, context.getRoot());
            return value;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}

```

### 四、测试数据

```java
import org.junit.Test;

import java.util.List;
import java.util.Map;

public class JsonCase {
    /**
     * {
     *     "a": {
     *         "b": {
     *             "c": {
     *                 "d": {
     *                     "e": "nothing"
     *                 }
     *             }
     *         }
     *     }
     * }
     */
    /**
     * 超多层级JSON嵌套的快速提取
     */
    @Test
    public void case0(){
        String text = "{\n" +
                "    \"a\": {\n" +
                "        \"b\": {\n" +
                "            \"c\": {\n" +
                "                \"d\": {\n" +
                "                    \"e\": \"nothing\"\n" +
                "                }\n" +
                "            }\n" +
                "        }\n" +
                "    }\n" +
                "}";
        Map<String, Object> jsonMap = JsonUtils.transferToMap(json);

        String e = JsonUtils.getValue(text, "a.b.c.d.e", String.class);
        System.out.println(e);
    }

    /**
     * {
     * "showapi_res_error": "",
     * "showapi_res_id": "628cc9850de3769f06edbb49",
     * "showapi_res_code": 0,
     * "showapi_fee_num": 1,
     * "showapi_res_body": {"ret_code":0,"area":"南安","areaid":"101230506","areaCode":"350583","hourList":[{"weather_code":"07","time":"202205242000","area":"南安","wind_direction":"东南风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"20"},{"weather_code":"07","time":"202205242100","area":"南安","wind_direction":"东风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"20"},{"weather_code":"07","time":"202205242200","area":"南安","wind_direction":"东风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"20"},{"weather_code":"07","time":"202205242300","area":"南安","wind_direction":"东风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"20"},{"weather_code":"07","time":"202205250000","area":"南安","wind_direction":"南风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"21"},{"weather_code":"07","time":"202205250100","area":"南安","wind_direction":"西风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"21"},{"weather_code":"07","time":"202205250200","area":"南安","wind_direction":"西北风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"21"},{"weather_code":"02","time":"202205250300","area":"南安","wind_direction":"西北风","wind_power":"0-3级 微风 <5.4m/s","weather":"阴","areaid":"101230506","temperature":"21"},{"weather_code":"02","time":"202205250400","area":"南安","wind_direction":"西北风","wind_power":"0-3级 微风 <5.4m/s","weather":"阴","areaid":"101230506","temperature":"21"},{"weather_code":"02","time":"202205250500","area":"南安","wind_direction":"西风","wind_power":"0-3级 微风 <5.4m/s","weather":"阴","areaid":"101230506","temperature":"21"},{"weather_code":"07","time":"202205250600","area":"南安","wind_direction":"西北风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"21"},{"weather_code":"07","time":"202205250700","area":"南安","wind_direction":"西北风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"21"},{"weather_code":"07","time":"202205250800","area":"南安","wind_direction":"西北风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"21"},{"weather_code":"07","time":"202205250900","area":"南安","wind_direction":"西南风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"22"},{"weather_code":"07","time":"202205251000","area":"南安","wind_direction":"南风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"23"},{"weather_code":"07","time":"202205251100","area":"南安","wind_direction":"东风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"24"},{"weather_code":"07","time":"202205251200","area":"南安","wind_direction":"东风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"24"},{"weather_code":"07","time":"202205251300","area":"南安","wind_direction":"东风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"25"},{"weather_code":"07","time":"202205251400","area":"南安","wind_direction":"东南风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"25"},{"weather_code":"07","time":"202205251500","area":"南安","wind_direction":"东南风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"25"},{"weather_code":"07","time":"202205251600","area":"南安","wind_direction":"东南风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"24"},{"weather_code":"07","time":"202205251700","area":"南安","wind_direction":"东南风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"23"},{"weather_code":"07","time":"202205251800","area":"南安","wind_direction":"东南风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"23"},{"weather_code":"07","time":"202205251900","area":"南安","wind_direction":"东南风","wind_power":"0-3级 微风 <5.4m/s","weather":"小雨","areaid":"101230506","temperature":"23"}]}
     * }
     */
    private String json = "{\n" +
            "\"showapi_res_error\": \"\",\n" +
            "\"showapi_res_id\": \"628cc9850de3769f06edbb49\",\n" +
            "\"showapi_res_code\": 0,\n" +
            "\"showapi_fee_num\": 1,\n" +
            "\"showapi_res_body\": {\"ret_code\":0,\"area\":\"南安\",\"areaid\":\"101230506\",\"areaCode\":\"350583\",\"hourList\":[{\"weather_code\":\"07\",\"time\":\"202205242000\",\"area\":\"南安\",\"wind_direction\":\"东南风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"20\"},{\"weather_code\":\"07\",\"time\":\"202205242100\",\"area\":\"南安\",\"wind_direction\":\"东风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"20\"},{\"weather_code\":\"07\",\"time\":\"202205242200\",\"area\":\"南安\",\"wind_direction\":\"东风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"20\"},{\"weather_code\":\"07\",\"time\":\"202205242300\",\"area\":\"南安\",\"wind_direction\":\"东风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"20\"},{\"weather_code\":\"07\",\"time\":\"202205250000\",\"area\":\"南安\",\"wind_direction\":\"南风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"21\"},{\"weather_code\":\"07\",\"time\":\"202205250100\",\"area\":\"南安\",\"wind_direction\":\"西风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"21\"},{\"weather_code\":\"07\",\"time\":\"202205250200\",\"area\":\"南安\",\"wind_direction\":\"西北风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"21\"},{\"weather_code\":\"02\",\"time\":\"202205250300\",\"area\":\"南安\",\"wind_direction\":\"西北风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"阴\",\"areaid\":\"101230506\",\"temperature\":\"21\"},{\"weather_code\":\"02\",\"time\":\"202205250400\",\"area\":\"南安\",\"wind_direction\":\"西北风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"阴\",\"areaid\":\"101230506\",\"temperature\":\"21\"},{\"weather_code\":\"02\",\"time\":\"202205250500\",\"area\":\"南安\",\"wind_direction\":\"西风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"阴\",\"areaid\":\"101230506\",\"temperature\":\"21\"},{\"weather_code\":\"07\",\"time\":\"202205250600\",\"area\":\"南安\",\"wind_direction\":\"西北风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"21\"},{\"weather_code\":\"07\",\"time\":\"202205250700\",\"area\":\"南安\",\"wind_direction\":\"西北风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"21\"},{\"weather_code\":\"07\",\"time\":\"202205250800\",\"area\":\"南安\",\"wind_direction\":\"西北风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"21\"},{\"weather_code\":\"07\",\"time\":\"202205250900\",\"area\":\"南安\",\"wind_direction\":\"西南风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"22\"},{\"weather_code\":\"07\",\"time\":\"202205251000\",\"area\":\"南安\",\"wind_direction\":\"南风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"23\"},{\"weather_code\":\"07\",\"time\":\"202205251100\",\"area\":\"南安\",\"wind_direction\":\"东风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"24\"},{\"weather_code\":\"07\",\"time\":\"202205251200\",\"area\":\"南安\",\"wind_direction\":\"东风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"24\"},{\"weather_code\":\"07\",\"time\":\"202205251300\",\"area\":\"南安\",\"wind_direction\":\"东风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"25\"},{\"weather_code\":\"07\",\"time\":\"202205251400\",\"area\":\"南安\",\"wind_direction\":\"东南风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"25\"},{\"weather_code\":\"07\",\"time\":\"202205251500\",\"area\":\"南安\",\"wind_direction\":\"东南风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"25\"},{\"weather_code\":\"07\",\"time\":\"202205251600\",\"area\":\"南安\",\"wind_direction\":\"东南风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"24\"},{\"weather_code\":\"07\",\"time\":\"202205251700\",\"area\":\"南安\",\"wind_direction\":\"东南风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"23\"},{\"weather_code\":\"07\",\"time\":\"202205251800\",\"area\":\"南安\",\"wind_direction\":\"东南风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"23\"},{\"weather_code\":\"07\",\"time\":\"202205251900\",\"area\":\"南安\",\"wind_direction\":\"东南风\",\"wind_power\":\"0-3级 微风 <5.4m/s\",\"weather\":\"小雨\",\"areaid\":\"101230506\",\"temperature\":\"23\"}]}\n" +
            "}";

    //将JSON转为标准Map结构
    @Test
    public void case1(){
        Map<String, Object> jsonMap = JsonUtils.transferToMap(json);
        System.out.println(jsonMap);
    }
    /**
     * OGNL直接提取数据，Value为子JSON对象的情况
     */
    @Test
    public void case2(){
        Map<String, Object> jsonMap = JsonUtils.transferToMap(json);
        Map resBody = JsonUtils.getValue(json, "showapi_res_body", Map.class);
        System.out.println(resBody);
    }

    /**
     * OGNL直接提取数据，Value为标准字符串的情况
     */
    @Test
    public void case3(){
        Map<String, Object> jsonMap = JsonUtils.transferToMap(json);
        String value = JsonUtils.getValue(json, "showapi_res_body.area", String.class);
        System.out.println(value);
    }

    /**
     * OGNL直接提取数据，Value为标准字符串的情况，Value为数组的情况
     */
    @Test
    public void case4(){
        Map<String, Object> jsonMap = JsonUtils.transferToMap(json);
        List<Map> hourList = JsonUtils.getValue(json, "showapi_res_body.hourList", List.class);
        System.out.println(hourList);
        // 每一个集合对象都是List
        for(Map hour : hourList){
            System.out.println(hour);
        }
    }

    /**
     * 利用List语法获取第6个时点天气预报
     */
    @Test
    public void case5(){
        Map<String, Object> jsonMap = JsonUtils.transferToMap(json);
        String area = JsonUtils.getValue(json, "showapi_res_body.hourList[5].weather_code", String.class);
        System.out.println(area);
    }
}

```

