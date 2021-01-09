springboot搭建访客管理系统
=

程序有问题，找[程序帮](http://ll032.cn/HZ6vHa)：QQ1022287044


项目介绍
----
>springboot搭建的访客管理系统，针对高端基地做严格把控来访人员信息管理，用户后端可以设置多个管理员帐号，给予不同部门的管理层使用，用户管理可以增加/修改内部成员的基本信息，需要到访的人员必须通过进入程序，在访客预约里面提交预约申请，预约后管理员可查询预约记录以及访客出入记录。

项目适用人群
----
正在做毕设的学生，或者需要项目实战练习的Java学习者

开发环境
-----
1. jdk 8
2. intellij idea
3. tomcat 8.5.40
4. mysql 5.7

所用技术
-----
1. springboot
2. mybatis
3. layUi 
4. JSP

项目访问地址
---
```
http://localhost:8090
帐号:admin 密码:admin
```


项目截图
----

 -  登录
 
 ![登录](/src/image/登录.png)

 -  子账号管理

 ![子账号管理](/src/image/子帐号管理.png)

 -  新增成员

 ![新增成员](/src/image/新增用户信息.png)

 -  预约列表

 ![预约列表](/src/image/预约列表.png)

 -  历史预约

 ![历史预约](/src/image/历史预约.png)

 -  出入影像记录

 ![出入影像记录](/src/image/出入影像记录.png)

 -  表格导出

 ![表格导出](/src/image/表格导出.png)

   - 访客预约申请

 ![访客预约申请](/src/image/预约申请.png)




关键代码:
-----
1. 用户信息
```diff 
public class SmartUser {
	@ApiModelProperty(value="用户编号",dataType="String",name="password")
	private Long id;
	@ApiModelProperty(value="登录帐号",dataType="String",name="account")
	private String account;
	@ApiModelProperty(value="用户名称",dataType="String",name="name")
	private String name;
	@ApiModelProperty(value="用户年龄",dataType="Integer",name="age")
	private int age;
	@ApiModelProperty(value="手机号",dataType="String",name="phone")
	private String phone;
	@ApiModelProperty(value="密码",dataType="String",name="password")
	private String password;
	@ApiModelProperty(value="mac",dataType="String",name="mac")
	private String mac;
	@ApiModelProperty(value="备注",dataType="String",name="remark")
	private String remark ;
	@ApiModelProperty(value="创建时间",dataType="String",name="createTime")
	private String createTime;
	private String headPic;
}
```

2. 添加访客记录
```diff
@ApiOperation(value="添加预约",notes="添加预约")
@ResponseBody
@PostMapping("/addVisitor")
public Response<String> addVisitor(Visitor visitor){
    SmartUser smartUser=new SmartUser();
    smartUser.setPhone(visitor.getUserPhone());
    smartUser.setName(visitor.getUserName());
    smartUser=smartUserService.login(smartUser);
    if(null!=smartUser){
        return visitorService.saveOrUpdate(visitor);
    }else{
        return Response.error(300);//查无一人
    }
}
```
 
3. 访客记录导出
```
@GetMapping("/exportExcel")
public void exportExcel(HttpServletResponse response) {
    try{
        List<List<String>> rows =new ArrayList<>();
        List<String> row1 = CollUtil.newArrayList("访客姓名", "访客手机号", "被访人姓名", "被访人电话", "预约日期", "访问事由");
        rows.add(row1);
        List<VisitorRecord>	list=smartUserService.getAll();
        for(VisitorRecord vr:list){
            rows.add(CollUtil.newArrayList(vr.getVisitorName(),  vr.getPhone(),vr.getUserPhone(),vr.getUserName(),vr.getAppointmentTime(),vr.getReasons()));
        }
        ExcelWriter writer = ExcelUtil.getWriter();
        writer.write(rows);
        response.setContentType("application/vnd.ms-excel;charset=utf-8");
        response.setHeader("Content-Disposition","attachment;filename="+ DateUtils.getTime3()+"visitorRecord.xls");
        ServletOutputStream out=response.getOutputStream();
        writer.flush(out);
        writer.close();
        IoUtil.close(out);
    }catch (Exception e){
        e.printStackTrace();
    }
}

```

4.过期预约做定时清理
```diff
@Scheduled(cron = "0 0/1 * * * ?")
private void configureTasks() {
    List<Visitor>  list=visitorService.findVisitorList("");
    if(list.size()>0){
        for(Visitor v:list){
            Long now=Long.valueOf(DateUtils.getTime2());
            Long appointmentTime=Long.valueOf(v.getAppointmentTime().replaceAll("-","").replaceAll(" ",""));
            if(appointmentTime-now<=0){
                VisitorRecord visitorRecord=new VisitorRecord();
                BeanUtils.copyProperties(v,visitorRecord);
                visitorRecordService.save(visitorRecord);
                visitorService.deleteUserById(Long.valueOf(v.getId()));
            }
        }
    }
}

``` 

注意事项
----
1. 预约地址需要有管理端分享地址给房主，由房主分享给到访的做预约登记
2. 后期增加房主端，新增房主查看记录
备注： 基础版做的比较简单，有条件的同学可以对接硬件设备，跑完整体流程，有问题可以和[程序帮](http://ll032.cn/HZ6vHa) 一起探讨
