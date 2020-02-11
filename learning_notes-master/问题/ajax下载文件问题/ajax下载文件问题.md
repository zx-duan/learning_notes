# Ajax下载文件问题



> 问题描述 : 使用ajax访问一个流数据的时候 发现ajax无法下载但是使用浏览器是可以访问的



解决办法:

今天遇到这样一个需求，将查查出来的数据导出来，不是将所有的数据导出来，而是要导出满足条件的数据，也就是说下载的时候要将查询的条件传到后台。

例如: 先查询课程性质是选修的课程然后导出来:

![img](ajax%E4%B8%8B%E8%BD%BD%E6%96%87%E4%BB%B6%E9%97%AE%E9%A2%98.assets/1196212-20180429155259849-833156467.png)

 

前台封装条件的form:

```html
<form class="layui-form layui-col-md12 x-so" id="queryCourseForm">
            <%--隐藏两个，一个当前页，一个页号--%>
                <%--当前页--%>
                <input type="hidden" name="pageNum"/>
                <input type="hidden" name="pageSize"/>

           <div class="layui-input-inline">
                <input type="text" name="coursenamecn"  placeholder="请输入课程中文名称" autocomplete="off" class="layui-input">
            </div>
            <div class="layui-input-inline">
                <select name="courseplatform">
                    <option value="">请选择课程平台</option>
                    <option value="通识教育">通识教育</option>
                    <option value="学科基础课">学科基础课</option>
                    <option value="专业课程">专业课程</option>
                    <option value="个性培养">个性培养</option>
                    <option value="教学环节">教学环节</option>
                </select>
            </div>

            <div class="layui-input-inline">
                <select name="coursenature">
                    <option value="">请选择课程性质</option>
                    <option value="必修">必修</option>
                    <option value="选修">选修</option>
                </select>
            </div>
            <div class="layui-input-inline">
                <select name="credit">
                    <option value="">请选择课程学分</option>
                    <option value="0-2">0-2</option>
                    <option value="2-4">2-4</option>
                    <option value="4-1000">4分以上</option>
                </select>
            </div>

            <button class="layui-btn" type="button" onclick="queryCourseFYBtn()"><i class="layui-icon">&#xe615;</i></button>
        </form>
```



后台springMCV接收查询条件并且生成一个excel文件到本地并且提供下载:



```java
package cn.xm.jwxt.controller.trainScheme;

import cn.xm.jwxt.service.trainScheme.CourseBaseInfoService;
import cn.xm.jwxt.utils.*;
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import org.apache.log4j.Logger;
import org.apache.poi.hssf.usermodel.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.*;
import java.lang.reflect.Method;
import java.net.URLEncoder;
import java.sql.SQLException;
import java.util.Date;
import java.util.List;
import java.util.Map;

/**
 * @Author: qlq
 * @Description 导出课程信息到Excel中
 * @Date: 10:11 2018/4/29
 */
@Controller
public class ExtCourseExcel {

    @Autowired
    private CourseBaseInfoService courseBaseInfoService;
    private Logger logger = Logger.getLogger(ExtCourseExcel.class);
    //1.先从缓存中取数据，看能取到取不到

    //2.写入excel到本地

    //3.打开流提供下载
    //1.查询数据
    public List<Map<String, Object>> getCourseBaseInfosByCondition(@RequestParam Map<String, Object> condition) {
        List<Map<String, Object>> datas = null;
        try {
            datas =  courseBaseInfoService.getCourseBaseInfosByCondition(condition);
        } catch (SQLException e) {
            logger.error("导出课程信息的时候查询数据库出错",e);
        }
        return datas;
    }


    //2.写文件到excel中

    /**
     * 写数据到本地磁盘
     * @param datas 课程数据
     * @param fileQualifyName   文件全路径(比如C:/USER/XXX.excel)
     */
    public void writeCourse2LocalExcel(List<Map<String,Object>> datas,String fileQualifyName){
        String[] title = { "序号", "课程编号", "课程平台","课程性质","中文名称","英文名称","学分/学时", "周学时分配","计分方式" };
        //2.1写入表头信息
        // 创建一个工作簿
        HSSFWorkbook workbook = new HSSFWorkbook();
        // 创建一个工作表sheet
        HSSFSheet sheet = workbook.createSheet();
        // 设置列宽
        this.setColumnWidth(sheet, 9);
        // 创建第一行
        HSSFRow row = sheet.createRow(0);
        // 创建一个单元格
        HSSFCell cell = null;
        // 创建表头
        for (int i = 0; i < title.length; i++) {
            cell = row.createCell(i);
            // 设置样式
            HSSFCellStyle cellStyle = workbook.createCellStyle();
            cellStyle.setAlignment(HSSFCellStyle.ALIGN_CENTER); // 设置字体居中
            // 设置字体
            HSSFFont font = workbook.createFont();
            font.setFontName("宋体");
            font.setBoldweight(HSSFFont.BOLDWEIGHT_BOLD);// 字体加粗
            // font.setFontHeight((short)12);
            font.setFontHeightInPoints((short) 13);
            cellStyle.setFont(font);
            cell.setCellStyle(cellStyle);
            cell.setCellValue(title[i]);
        }


        // 2.2写入数据
        // 从第二行开始追加数据
        for (int i = 1, length_1 = (datas.size() + 1); i < length_1; i++) {
            // 创建第i行
            HSSFRow nextRow = sheet.createRow(i);
            // 设置样式
            HSSFCellStyle cellStyle = workbook.createCellStyle();
            cellStyle.setAlignment(HSSFCellStyle.ALIGN_CENTER); // 设置字体居中
            // 获取数据(一条数据)
            Map<String, Object> course = datas.get(i - 1);
            for (int j = 0; j < 9; j++) {
                HSSFCell cell2 = nextRow.createCell(j);
                cell2.setCellStyle(cellStyle);
                if (j == 0) {
                    cell2.setCellValue(i);//第一列是序号
                    continue;
                }
                if (j == 1) {
                    cell2.setCellValue(course.get("courseNum").toString());//课程编号
                    continue;
                }
                if (j == 2) {
                    cell2.setCellValue(course.get("coursePlatform").toString());//课程平台
                    continue;
                }
                if (j == 3) {
                    cell2.setCellValue(course.get("courseNature").toString());//课程性质
                    continue;
                }
                if (j == 4) {
                    cell2.setCellValue(course.get("courseNameCN").toString());//中文名称
                    continue;
                }
                if (j == 5) {
                    cell2.setCellValue(course.get("courseNameEN").toString());//英文名称
                    continue;
                }
                if (j == 6) {
                    cell2.setCellValue(course.get("credit").toString()+"/"+course.get("courseHour").toString());//学分/学时
                    continue;
                }
                if (j == 7) {
                    cell2.setCellValue(course.get("weeklyHour").toString());//周学时
                    continue;
                }
                if (j == 8) {
                    cell2.setCellValue(course.get("scoringWay").toString());//计分方式
                    continue;
                }
            }
        }


        // 创建一个文件
        File file = new File(fileQualifyName);
        // 获取文件的父文件夹并删除文件夹下面的文件
        File parentFile = file.getParentFile();
        // 获取父文件夹下面的所有文件
        File[] listFiles = parentFile.listFiles();
        if (parentFile != null && parentFile.isDirectory()) {
            for (File fi : listFiles) {
                // 删除文件
                fi.delete();
            }
        }
        // 如果存在就删除
        if (file.exists()) {
            file.delete();
        }
        try {
            file.createNewFile();
            // 打开文件流并写入文件
            FileOutputStream outputStream = org.apache.commons.io.FileUtils.openOutputStream(file);
            workbook.write(outputStream);
            outputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 设置列宽的函数
     * @param sheet 对哪个sheet进行设置，
     * @param colNum
     */
    private  void setColumnWidth(HSSFSheet sheet, int colNum) {
        for (int i = 0; i < colNum; i++) {
            int v = 0;
            v = Math.round(Float.parseFloat("15.0") * 37F);
            v = Math.round(Float.parseFloat("20.0") * 267.5F);
            sheet.setColumnWidth(i, v);
        }
    }

    //3.打开流提供下载
    @RequestMapping("/downCourses")
    public void down(HttpServletRequest request, HttpServletResponse response,@RequestParam Map condition){
        //1.查询数据
        List<Map<String, Object>> datas = this.getCourseBaseInfosByCondition(condition);
        //2.写入excel
        String dir = ResourcesUtil.getValue("path","courseExcelFile");
        String fileName = DefaultValue.COURSE_DEFAULT_FILENAME;
        String fileQualifyName =  dir + fileName;//生成的excel名字
        this.writeCourse2LocalExcel(datas,fileQualifyName);//写入数据(生成文件)
        //3.打开流提供下载
        //获取输入流
        try {
            InputStream bis = new BufferedInputStream(new FileInputStream(new File(fileQualifyName)));
            fileName = URLEncoder.encode(fileName,"UTF-8");
            //设置文件下载头
            response.addHeader("Content-Disposition", "attachment;filename=" + fileName);
            //1.设置文件ContentType类型，这样设置，会自动判断下载文件类型
            response.setContentType("multipart/form-data");
            BufferedOutputStream out = new BufferedOutputStream(response.getOutputStream());
            int len = 0;
            while((len = bis.read()) != -1){
                out.write(len);
                out.flush();
            }
            out.close();
        } catch (Exception e) {
            logger.error("下载课程信息出错!",e);
        }
    }
}
```



接下来尝试用ajax的post提交表单进行下载:

```
$.ajax({
                url:contextPath+"/downCourses.do",
                type:'post',
                async:true,
                data:$("#queryCourseForm").serialize(),//携带查询条件
                success:function () {
                   layer.close(index);
                }
            });
```



结果:响应头正确，数据也正确的传到后台，但是未下载文件:

![img](ajax%E4%B8%8B%E8%BD%BD%E6%96%87%E4%BB%B6%E9%97%AE%E9%A2%98.assets/1196212-20180429155842657-1020261264.png)

 

 ![img](ajax%E4%B8%8B%E8%BD%BD%E6%96%87%E4%BB%B6%E9%97%AE%E9%A2%98.assets/1196212-20180429155926572-1100100136.png)

 **总结:**

　　即使ajax请求到一个controller在跳转到下载的controller上也不能下载，百度了一下总结下原因：发现原来jQuery的ajax回调已经把response的数据傻瓜式的以字符串的方式解析.

 

###  **解决办法:**

- 第一种:将传条件的以表单提交的方式进行(推荐这种)-----这种方式也可以用来页面跳转

```
            $("#queryCourseForm").attr("action",contextPath+"/downCourses.do");//改变表单的提交地址为下载的地址
            $("#queryCourseForm").submit();//提交表单
```

 

- 第二种:以window.location.href="xxx"的方式请求下载地址

```
            window.location.href=contextPath+"/downCourses.do"
```

　　这种方法需要自己手动的拼接地址传递参数。get请求携带参数的方式:  xxxx.html?username=xxx&password=xxxx

 

- 第三种:动态创建表单加到fbody中，最后删除表单(推荐这种,可以将组合条件的值也动态的加入表单中)

```javascript
    //动态创建表单加到fbody中，最后删除表单
    var queryForm = $("#queryCourseForm");
    var exportForm = $("<form action='/downCourses.do' method='post'></form>")     
    exportForm.html(queryForm.html());
    $(document.body).append(exportForm);
    exportForm.submit();
    exportForm.remove();
```



注意:动态form必须加到DOM树，否则会报异常:Form submission canceled because the form is not connected。而且提交完需要删除元素。

 

**进一步完善代码:**

```javascript
    function download() {
        try {
            var queryForm = $("#queryCourseForm");
            var exportForm = $("<form action='/downCourses.do' method='post'></form>")
            exportForm.html(queryForm.html());
            $(document.body).append(exportForm);
            exportForm.submit();
        } catch (e) {
            console.log(e);
        } finally {
            exportForm.remove();
        }
    }
```



 

补充:  有时候上面查询条件到不了第二个动态表单中，需要手动添加查询条件到表单中，如下:

```javascript
function extTaizhang(){
    //动态创建表单加到fbody中，最后删除表单
    var queryForm = $("#queryTaizhangForm");
    var exportForm = $("<form action='"+baseurl+"/extSafeHatTaizhang.do' method='post'></form>")     
    
    exportForm.append("<input type='hidden' name='userName' value='"+$("[name='userName']").val()+"'/>")
    exportForm.append("<input type='hidden' name='idCard' value='"+$("[name='idCard']").val()+"'/>")
    exportForm.append("<input type='hidden' name='safeHatNum' value='"+$("[name='safeHatNum']").val()+"'/>")
    alert(exportForm.serialize());
    $(document.body).append(exportForm);
    exportForm.submit();
    exportForm.remove(); 
}
```

 

进一步封装如下: (第一个参数是form元素的ID，也就是查询条件所在的form，第二个参数是下载的URL，会自动遍历条件并拼接查询条件)

```javascript
function exportExcel(formId, url) {
    try {
        var queryForm = $("#" + formId);
        var exportForm = $("<form action='" + url + "' method='post'></form>")
        
        queryForm.find("input").each(function() {
            var name = $(this).attr("name");
            var value = $(this).val();
            exportForm.append("<input type='hidden' name='" + name + "' value='" + value + "'/>")
        });
        
        queryForm.find("select").each(function() {
            var name = $(this).attr("name");
            var value = $(this).val();
            exportForm.append("<input type='hidden' name='" + name + "' value='" + value + "'/>")
        });
        
        $(document.body).append(exportForm);
        exportForm.submit();
    } catch (e) {
        console.log(e);
    } finally {
        exportForm.remove();
    }
}
```



调用代码:

```javascript
exportExcel('listPageForm', 'extOperationCharge.do')
```