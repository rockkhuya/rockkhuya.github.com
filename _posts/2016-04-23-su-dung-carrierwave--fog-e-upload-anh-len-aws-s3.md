---
layout: post
title: "Sử dụng CarrierWave + fog để upload ảnh lên trên AWS S3"
description: ""
category: Rails
tags: [CarrierWave, AWS, S3, upload]
comments: True
---
Khi sử dụng những dịch vụ PaaS như Heroku, có 1 vấn đề hay gặp phải là vấn đề upload ảnh, tài liệu, ... Một cách xử lý hay được sử dụng là sử dụng thêm dịch vụ AWS S3 để tạo nơi chứa ảnh, tài liệu, ...

Có nhiều gem để giúp Rails upload file lên trên server,  như CarrierWave, PaperClip, ... ở đây sẽ hướng dẫn cách sử dụng [CarrierWave](https://github.com/carrierwaveuploader/carrierwave). 

Ngoài carrierwave ra, còn cần thêm gem ""fog" nữa để hỗ trợ upload lên các server cloud. Các thông tin khác về fog. 

[fog : The Ruby cloud services library](http://fog.io/)

[GitHub : fog](https://github.com/fog/fog)


# Môi trường

||version|
|:--|:--|
|Ruby|2.1.1|
|Rails|4.2.0|
|CarrierWave|0.10.0|
|fog|1.28.0|


# Tạo tài khoản AWS

Tạo tài khoản AWS nếu chưa có. 

[Hướng dẫn tạo tài khoản AWS](http://aws.amazon.com/jp/register-flow/)


# Tạo bucket S3 và tài khoản IAM

Để lưu file trên S3, bạn cần phải tạo 1 bucket và 1 tài khoản IAM để access vào S3.  Hướng dẫn về cách tạo những thông tin này bạn có thể tham khảo trên amazon hoặc tìm trên mạng. 

# Install

## Gemfile
Thêm carrierwave và fog vào Gemfile

```ruby
gem 'carrierwave'
gem 'fog'
```

`$ bundle install --path vendor/bundle`
( thêm `--path` nếu bạn muốn tạo 1 project đóng )

## thay đổi CarrierWave uploader

Sửa `storage` thành `:fog`

```ruby
storage :fog
```


## Tạo file setting cho CarrierWave

Các setting cho carrierwave + fog để upload file lên trên S3. 

Setting theo các môi trường production, development, test sẽ thuận tiện hơn. 

config/initialize/carrierwave.rb

```ruby
CarrierWave.configure do |config|
  config.fog_credentials = {
    provider: 'AWS',
    aws_access_key_id: 'access_key',
    aws_secret_access_key: 'secret_key',
    region: 'ap-northeast-1' # set theo khu vực khi bạn tạo bucket
  }

  case Rails.env
    when 'production'
      config.fog_directory = 'dummy'
      config.asset_host = 'https://s3-ap-northeast-1.amazonaws.com/dummy'

    when 'development'
      config.fog_directory = 'dev.dummy'
      config.asset_host = 'https://s3-ap-northeast-1.amazonaws.com/dev.dummy'

    when 'test'
      config.fog_directory = 'test.dummy'
      config.asset_host = 'https://s3-ap-northeast-1.amazonaws.com/test.dummy'
  end
end
```

`aws_access_key` , `aws_secret_access_key`  có được khi bạn tạo tài khoản IAM. 
Về region bạn tham khảo thêm ở đây [aws documentation region and endpoint](http://docs.aws.amazon.com/ja_jp/general/latest/gr/rande.html#s3_region)

`fog_directory` là tên bucket trên S3.

`asset_host` là endpoint bạn muốn lấy ra.  Bạn có thể viết như trên, hoặc dùng trực tiếp domain của mình cũng được. 

Chúc bạn thành công !

