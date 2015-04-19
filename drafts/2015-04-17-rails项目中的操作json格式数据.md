<!-- ---
layout: post
title:  "rails项目中的操作json格式数据"
date:   2015-04-17 11:41:00
categories: rails json
comments: true
---

1.rails中操作json数据是经常的事情，怎么样简单方便的去操作，jbuilder给出了答案。
2.普通方法： rails的action中：
class PostsController < ApplicationController
   def index
      @posts = Post.all
       respond_to do |format|
          format.html
          format.json {render json: @posts}  #json返回对象的所有字段包括时间戳
   end
end
这样我们请求/posts.json会返回json格式的数据。
注：使用render json返回所有的字段，我们无法定制，所有就出来了to_json这个方法
format.json {render json: @posts.to_json(only: [:title])}  #指定返回title字段
如果我们需要请求/posts返回json对象直接用: render json:  @posts.to_json
3.jbuilder方法：随着项目复杂性的增加，管理json数据将变得越来越复杂，所以jbuilder是个比较好的选择。jbuilder只是一个模板引擎。我们并不想在每个action中都写format响应格式。比如show中只有@post = Post.find(params[:id]),这样使用jbuilder模板就方便很多。如创建：show.json.jbuilder
json.id @posts.ids
json.title @posts.title
注：这种写法在字段比较多时可用extract!方法替换：json.extract! @post, :id, :title, :created_at
另一种选择是： json.(@post, :id, :title, created_at)
将show action的format去除
  def show
    @post= Post.find(params[:id])
  end
 这样我们请求/posts/1.json时返回：{"id":1,"title":"the first blog"}  只包括id和title两个字段，并且action中也不用那么冗杂。

 https://github.com/rails/jbuilder
jbuilder是一个简化json书写的DSL（区域专属语言）。在rails中不用编写很多大括号的代码，直接使用jbuilder模板渲染即可。
如: app/views/message/show.json.jbuilder
 -->