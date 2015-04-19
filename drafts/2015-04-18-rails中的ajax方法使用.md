<!-- ---
layout: post
title:  "rails中如何使用ajax"
date:   2015-04-18 12:11:00
categories: rails ajax
comments: true
---

1.rails提供的ajax方法：

link_to 'xxx', url, remote: true    #rails默认将普通链接转化为ajax链接
form_for(@post, remote: true)  #将表单转化为ajax提交，   相同有： form_tag(url, remote: true)
以上两种方法由ujs提供

例子：
link_to "new",  new_post_path, remote: true   rails将该链接识别为ajax请求， 自动请求到 new.js.erb, 在new.js.erb输入如下内容：$("some_element").replaceWith("<%= j render 'some/template'%>")   即该点击会将some_element中加载render的页面内容， 其中j是escape_javascript的别称
 


2. 为某个对象创建api
def api
    @products = Product.all
    render json: @products.to_json(only: [:title, :id, :number])
  end
在路由中： 
collection do 
          get :api
  end
这样我们在请求/products/api时会返回product对象的title, id, number的数据。
to_json方法： 
@product.to_json(only: [:id, :name])
@product.to_json(except: [:id, :name])
操作关联对象：
@product.to_json(include: :product_images)


前言： 日常基础性验证，使用Jquery的validation插件就可以完成，如果浏览器支持html5的话，直接使用html5自带验证即可。但是很多操作需要浏览器和服务器端双重验证，比如支付或者下单类的判断信息。rails结合ajax实现客户端，浏览器端双重验证方法：
在控制器中定义一个方法：
def create
    @user = User.find_by(code: params[:user][:code])
    if @user.blank?
      return render js: "$('span#js_user_code').html('*赠送的用户不存在！');"  #第一种方式在表单后面提示验证信息。
      #return render js: 'alert("赠送的用户不存在！");'  #第二种是alert信息即js弹出框信息。
    end
   #邮箱格式判断
     if /^(\w-*\.*)+@(\w-?)+(\.\w{2,})+$/.match(params[:site_user][:email]).nil?
          return render js: "$('#error-info').html('* 邮箱格式不正确');"
      end
   #
   if @user.save
     return render js: 'alert("赠送成功！");window.location(history.go(0));'  #弹出成功提示框，确认后刷新当前页面。remote导致redirect_to失效，所以这样处理。
    return render js: "window.location.href = '#{www_products_url}';" 
  else
        return render js: 'alert("赠送失败！");'
  end
end
表单中：
= form_for(@user, remote: true) do |f|  #在form中指定remote: true即ajax提交方式。
  table
    tr
      th 用户：
      td 
        = text_field_tag :pactera_user_code
        span#js_pactera_user_code  #显示报错信息的占位
效果：

下面是用ajax的纯客户端验证方法，实现一般的验证是可以的，但是若用户禁掉js就会使验证失效，所以还是推荐使用上面方法。
javascript:
  $(function(){
    $('#pactera_user_code').blur(function() {  #表单失去焦点事件
      var _this = $(this); #获取表单中输入的值，存到_this变量中
      $.get(
        '#{my_verify_pactera_user_path}',  #jquery中$.get()方法的第一个参数url
        { pactera_user_code: _this.val() },  #jquery中$.get()方法的第二个参数，即需要传给服务器的参数，严格采用key/value方式
        function(data) {   #参数data获得前面参数的值，如果多个值可用data. pactera_user_code获取表单中输入的值。
          if(data == null) {
            alert('用户不存在!');
            _this.val('');
            _this.focus();
          } else if(data.code == '#{current_pactera_user.code}') {
            alert('不能赠送给自己!');
            _this.val('');
            _this.focus();
          }
        }
      );
    });
  });
 -->