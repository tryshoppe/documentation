# Attachments

Shoppe has support for uploading attachments to many cloud services
such as S3, Cloud Files and more. Because Shoppe uses CarrierWave and Fog,
2 popular gems to file uploading, this makes it simple to upload
to different services.

You can see more information on the [CarrierWave GitHub readme](https://github.com/carrierwaveuploader/carrierwave#fog).

The example below uploads to S3 in Production and uses standard file
hosting in development. Change the options as required.

```ruby
::config/initializers/carrierwave.rb
CarrierWave.configure do |cfg|
  if Rails.env.production?
    cfg.storage = :fog
    cfg.fog_credentials = {
      :provider               => 'AWS',
      :region                 => "eu-west-1",
      :aws_access_key_id      => ENV["AWS_ACCESS_ID"],
      :aws_secret_access_key  => ENV["AWS_ACCESS_SECRET"]
    }
    cfg.fog_directory  = ENV["AWS_BUCKET"]
    cfg.fog_public     = true
    cfg.fog_attributes = {'Cache-Control'=>'max-age=315576000'}
  else
    cfg.storage = :file
  end

  cfg.enable_processing = !Rails.env.test?
end
```

## Converting nifty-attachments to Shoppe Attachments

Shoppe v1.0.9 moved from `nifty-attachments` to a custom Attachments system.
Once installing the latest version and running the migrations, you'll notice 
that none of the old attachments have been moved over. In order to run this, you'll
need to add the `nifty-attachments` gem to your Gemfile. It can then be
removed afterwards. Start the conversion by running the following rake task:

```bash
$ rake shoppe:attachments
```