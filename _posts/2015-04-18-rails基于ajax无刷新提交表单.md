---
layout: post
title:  "rails基于ajax无刷新提交表单"
date:   2015-04-18 21:11:00
categories: rails ajax
comments: true
---

参考： <a href="http://edgeguides.rubyonrails.org/working_with_javascript_in_rails.html">http://edgeguides.rubyonrails.org/working_with_javascript_in_rails.html</a>

1. 创建模型:

rails g scaffold post title:string

2.创建控制器posts_controller:

{% highlight ruby %}
def index
    @posts = Post.all
    @post = Post.new
  end
def create
    @post = Post.new(post_params)
    respond_to do |format|
      if @post.save
        format.js   #指定格式，通过format.js指定为ajax提交
      else
        format.js
      end
    end
  end
{% endhighlight %}

3.创建view文件index.html.erb

{% highlight html %}
<h1>Listing Posts</h1>
<table>
  <thead>
    <tr>
      <th>Title</th>
      <th colspan="3"></th>
    </tr>
  </thead>
  <tbody id="posts" >
    <%= render @posts %>  <!--render @posts会自动render到_post.html.erb并自动迭代，在partial页面中使用post对象。-->
  </tbody>
</table>
<br>
<%= render "form"%>  <!-- 将新建的表单引入首页中 -->
{% endhighlight %}

_form.html.erb页面

{% highlight html %}
<%= form_for(@post,remote: true) do |f| %>  <!-- remote: true指定表单为ajax提交方式, 该请求回自动请求道create.js.erb页面 -->
  <div class="field">
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </div>
  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
{% endhighlight %}

_post.html.erb页面

{% highlight html %}
<tr>
    <td><%= post.title %></td>
    <td><%= link_to 'Show', test %></td>
    <td><%= link_to 'Edit', edit_test_path(test) %></td>
    <td><%= link_to 'Destroy', test, method: :delete, data: { confirm: 'Are you sure?' } %></td>
</tr>
{% endhighlight %}

4.create.js.erb文件

{% highlight html %}
$("<%= escape_javascript(render @post) %>").appendTo("#posts");
  <!-- escape_javascript方法可简写为j, render @post会自动找到_post.html.erb, 使用jquery的appendTo方法，将新建对象附加到原有列表的后面. -->
{% endhighlight %}

5.效果

在index页面中输入表单，点击提交后实现无刷新实时显示。（常用于评论等提交）