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
      Users.create! row.to_hash
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
<%= form_tag home_path, multipart: true do %>
  <%= file_field_tag :file %>
  <%= submit_tag "Import CSV" %>
<% end %>
{% endhighlight %}
routes.rb文件：
{% highlight ruby %}
  resources :products do
    member do
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
require 'csv'
def self.import(file)
   allowed_attributes = [ "id","name","quantity","price","number"]
    spreadsheet = open_spreadsheet(file)
    header = spreadsheet.row(1)
    (2..spreadsheet.last_row).each do |i|
      row = Hash[[header, spreadsheet.row(i)].transpose]
      product = find_by_id(row["id"]) || new
      product.attributes = row.to_hash.select { |k,v| allowed_attributes.include? k }
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

参考： <a href="http://richonrails.com/articles/importing-csv-files">http://richonrails.com/articles/importing-csv-files</a>
<a href="http://railscasts.com/episodes/396-importing-csv-and-excel">http://railscasts.com/episodes/396-importing-csv-and-excel</a>
<a href="https://github.com/roo-rb/roo">https://github.com/roo-rb/roo</a>
