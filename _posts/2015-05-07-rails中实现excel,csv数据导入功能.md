---
layout: post
title:  "rails中实现excel,csv数据导入功能"
date:   2015-05-07 17:20:00
categories: rails excel
comments: true
---
在rails中导入csv文件比较简单，使用ruby自带的csv库即可完成..

详见<a href="http://ruby-doc.org/stdlib-1.9.3/libdoc/csv/rdoc/CSV.html">http://ruby-doc.org/stdlib-1.9.3/libdoc/csv/rdoc/CSV.html</a>

以product表操作为例:

product.rb文件:
{% highlight ruby %}
  require 'csv' #注意必须小写
  def self.import(file)
    CSV.foreach(file.path, headers: true) do |row|
      Product.create! row.to_hash
    end
  end
{% endhighlight %}
products_controller.rb文件:
{% highlight ruby %}
def import
  Product.import(params[:file])
end
{% endhighlight %}
view文件:
{% highlight ruby %}
<%= form_tag import_products_path, multipart: true do %>
  <%= file_field_tag :file %>
  <%= submit_tag "Import CSV" %>
<% end %>
{% endhighlight %}
routes.rb文件：
{% highlight ruby %}
  resources :products do
    collection do
      post :import
    end
  end
{% endhighlight %}
通过以上配置就可实现csv文件的导入了．


而通过excel文件将数据导入数据库是通过roo插件来实现的:

Gemfile文件:
{% highlight ruby %}
gem "roo"
{% endhighlight %}

配置控制器products_controller.rb:
{% highlight ruby %}
def import
  Product.import(params[:file])
  redirect_to root_url, notice: "商品导入成功!."
end
{% endhighlight %}

在model文件中定义import方法product.rb:
{% highlight ruby %}
require 'csv'  #如果需要引入csv的地方比较多可把这句提到config/application.rb中
def self.import(file)
   allowed_attributes = [ "id","name","quantity","price","number"]
    spreadsheet = open_spreadsheet(file)  #使用open_spreadsheet方法读取文件
    header = spreadsheet.row(1)  #获得excel文件的第一行，保存到header变量
    (2..spreadsheet.last_row).each do |i|  #从第二行开始循环
      row = Hash[[header, spreadsheet.row(i)].transpose]  #创建header中字段名为key,excel单元格内容为value的hash
      product = find_by_id(row["id"]) || new  #row["id"]返回单元格中的id值，可以用其他值如row["number"]
      product.attributes = row.to_hash.select { |k,v| allowed_attributes.include? k }  #select生成新的hash，并对应赋值.如果有单个值不能用这种赋值方法，可以先初始化u=row.to_hash.select { |k,v| allowed_attributes.include? k },再单独赋值u["number"]="xxx",再全部赋值：product.attributes=u.
      product.save!
    end
  end

  def self.open_spreadsheet(file)
    case File.extname(file.original_filename)
    when ".csv" then Roo::CSV.new(file.path)
    when ".xls" then Roo::Excel.new(file.path)
    when ".xlsx" then Roo::Excelx.new(file.path)
    else raise "未知格式: #{file.original_filename}"
    end
  end
{% endhighlight %}

update -- 2015-05-26 --
最近做了个关联表导入创建的功能，关系是has_many, 一个question关联多个option,例如一个题目对应4个选项的关系。最好加上事务处理实现非法数据自动回滚。现在验证方面还有问题有待研究。

{% highlight ruby %}
def self.import(file)
    allowed_attributes = [ "title", "signal_score", "correct_option", "correct_hint"]
    spreadsheet = open_spreadsheet(file)
    header = spreadsheet.row(2)
    (3..spreadsheet.last_row).each do |i|   #从表格的第三行（包括第三行）开始迭代
      row = Hash[[header, spreadsheet.row(i)].transpose] #transpose方法要好好看一下
      question = Question.new
      question.attributes = row.to_hash.select { |k,v| allowed_attributes.include? k }
      question.save!
      option_length = row.length - 4  #行中减去question选项长度,即option的长度
      (1..option_length).each do |i|
        _opt = 'option_' + i.to_s
        option = question.options.new
        option.name = row[_opt]
        option.save!
      end
    end
  end
{% endhighlight %}

数据结构：
![alt tag](http://ww3.sinaimg.cn/bmiddle/738e4c72jw1esi17gpucij20jb02ct9j.jpg)

参考： <a href="http://richonrails.com/articles/importing-csv-files">http://richonrails.com/articles/importing-csv-files</a>
<a href="http://railscasts.com/episodes/396-importing-csv-and-excel">http://railscasts.com/episodes/396-importing-csv-and-excel</a>
<a href="https://github.com/roo-rb/roo">https://github.com/roo-rb/roo</a>
