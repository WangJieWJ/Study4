# Study4
评论互动功能实现

## Season代码
1、Season启动类：APP.java
```java
package com.trs.config;
import com.season.core.spring.SeasonApplication;
import com.season.core.spring.SeasonRunner;

/**
 * Created by wangjie on 2016/10/14 0014.
 */
public class App extends SeasonApplication {

    public static void main(String[] args){
        SeasonRunner.run(App.class,args);
    }

}
```

2、配置启动类：ServletInitializer.java
```java
package com.trs.config;

import com.season.core.spring.SeasonServletInitializer;

/**
 * Created by wangjie on 2016/10/14 0014.
 */
public class ServletInitializer extends SeasonServletInitializer{
    @Override
    protected Class<?> getAppClass() {
        return App.class;
    }
}
```
3、处理请求：TestController.java
```java
package com.trs.controller;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.season.core.Controller;
import com.season.core.ControllerKey;
import com.trs.domain.Review;
import com.trs.service.ReviewService;

import java.util.List;

/**
 * Created by wangjie on 2016/10/14 0014.
 */
@ControllerKey("hello")
public class TestController extends Controller {


    public void save() {
        String RName = getPara("RName");
        String RContent = getPara("RContent");
        String URL=getPara("URL");
        Review review=new Review(RName,RContent);
        ReviewService reviewService=new ReviewService();
        reviewService.saveReview(review);

        renderText("存储成功！");   
        //调用方式：http://localhost:8080/hello/save?RName=WJ&RContent=%E5%86%85%E5%AE%B9%E7%95%8C%E9%9D%A2&URL=asdasdasdasd
    }

    public void get() {
        String RName=getPara("RName");
        String URL=getPara("URL");
        ReviewService reviewService=new ReviewService();
        List<Review> lists=reviewService.getReviewByName(RName);

        JSON json= (JSON) JSONArray.toJSON(lists);

        renderText(json.toJSONString());
    }
}
```

4、数据库层：ReviewDao.java
```java
package com.trs.dao;

import com.trs.domain.Review;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

/**
 * Created by wangjie on 2016/10/15 0015.
 */
public class ReviewDao {

    public List<Review> getReviewsByName(String Rname) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet RS = null;
        try {
            Class.forName("com.mysql.jdbc.Driver");
            connection = DriverManager.getConnection("jdbc:mysql:///TRSWCMV7", "root", "MySQL_1234");
            String sql = "SELECT * FROM review where RNAME=?";
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setString(1, Rname);
            RS = preparedStatement.executeQuery();
            List<Review> lists = new ArrayList<Review>();
            while (RS.next()) {
                lists.add(new Review(RS.getString(2), RS.getString(3), RS.getDate(4)));
            }
            return lists;
        } catch (Exception e) {
            return null;
        } finally {
            try {
                if (RS != null)
                    RS.close();
                if (preparedStatement != null)
                    preparedStatement.close();
                if (connection != null)
                    connection.close();
            } catch (SQLException e) {
            }
        }
    }

    public void saveReview(Review review) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        try {
            Class.forName("com.mysql.jdbc.Driver");
            connection = DriverManager.getConnection("jdbc:mysql:///TRSWCMV7", "root", "MySQL_1234");
            String sql = "INSERT INTO review(RNAME,RCONTENT) values(?,?)";
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setString(1, review.getRNAME());
            preparedStatement.setString(2, review.getCONTENT());
            preparedStatement.execute();
        } catch (Exception e) {
        }
    }
}
```

5、实体类：Review.java
```java
package com.trs.domain;


import java.sql.Date;

/**
 * Created by wangjie on 2016/10/14 0014.
 * 实体类
 */
public class Review {

    private int RID;
    private String RNAME;
    private String CONTENT;
    private Date RDATE;     //向数据库插入的时候，使用触发器。

    public int getRID() {
        return RID;
    }

    public void setRID(int RID) {
        this.RID = RID;
    }

    public String getRNAME() {
        return RNAME;
    }

    public void setRNAME(String RNAME) {
        this.RNAME = RNAME;
    }

    public String getCONTENT() {
        return CONTENT;
    }

    public void setCONTENT(String CONTENT) {
        this.CONTENT = CONTENT;
    }

    public Date getRDATE() {
        return RDATE;
    }

    public void setRDATE(Date RDATE) {
        this.RDATE = RDATE;
    }

    public Review(String RNAME, String CONTENT) {
        this.RNAME = RNAME;
        this.CONTENT = CONTENT;
    }

    public Review(String RNAME, String CONTENT,Date DATE) {
        this.RNAME = RNAME;
        this.CONTENT = CONTENT;
        this.RDATE=DATE;
    }


    @Override
    public String toString() {
        return "Review{" +
                "RID=" + RID +
                ", RNAME='" + RNAME + '\'' +
                ", CONTENT='" + CONTENT + '\'' +
                ", RDATE=" + RDATE +
                '}';
    }

    public Review() {
    }
}
```

6、业务层：ReviewService.java  
```java
package com.trs.service;

import com.trs.dao.ReviewDao;
import com.trs.domain.Review;

import java.util.List;

/**
 * Created by wangjie on 2016/10/14 0014.
 */
//@Repository
public class ReviewService {

//    @Autowired
//    private DataSource dataSource;
//
//    private JdbcTemplate jdbcTemplate;
//
//    @Autowired
//    public void setJdbcTemplate(JdbcTemplate jdbcTemplate){
//        jdbcTemplate.setDataSource(dataSource);
//        this.jdbcTemplate=jdbcTemplate;
//    }
//
//    public List<Review> getList(){
//        String sql="SELECT * FROM Review";
//        return jdbcTemplate.queryForList(sql,Review.class);
//    }
//
//    public void save(String RNAME,String RCONTENT){
//        String sql="INSERT INTO Review(RNAME,RCONTENT) values("+RNAME+","+RCONTENT+")";
//        jdbcTemplate.execute(sql);
//    }

    private ReviewDao reviewDao;

    public ReviewService(){
        this.reviewDao=new ReviewDao();
    }

    /***
     * 按照名称查找所有评论
     * @param Rname
     * @return
     */
    public List<Review> getReviewByName(String Rname){
        return reviewDao.getReviewsByName(Rname);
    }

    /**
     * 保存评论
     * @param review
     */
    public void saveReview(Review review){
        reviewDao.saveReview(review);
    }

}
```
