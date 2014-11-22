target_bucket = "tartaneaglesgonorth.com"

desc "Deploy site to Amazon S3"
task :deploy do
  require "s3"
  require "mime/types"

  id = ENV["TARTAN_EAGLES_ACCESS_KEY_ID"]
  key = ENV["TARTAN_EAGLES_SECRET_ACCESS_KEY"]

  if id.nil? || key.nil?
    puts "S3 credentials not found in environment."
    exit 1
  end

  service = S3::Service.new(:access_key_id => id, :secret_access_key => key)
  bucket = service.buckets.find(target_bucket)

  local_files = Dir["_site/**/*"].map { |file| file.sub(/_site\//, "") }
  remote_files = bucket.objects

  puts "Checking for local files to upload..."
  changes = false
  local_files.each do |local_file|
    next if File.directory? "_site/#{local_file}"
    files = remote_files.select { |remote_file| remote_file.key == local_file }
    remote_file = files.first

    if remote_file
      local_md5 = Digest::MD5.file("_site/#{local_file}").hexdigest
      remote_md5 = remote_file.etag

      unless local_md5 == remote_md5
        remote_file.content = open("_site/#{local_file}")
        remote_file.content_type = MIME::Types.of(local_file).first.content_type
        remote_file.save
        puts " |  #{local_file} has changed."
        changes = true
      end
    else
      local_mime = MIME::Types.of(local_file).first
      remote_file = bucket.objects.build(local_file)
      remote_file.content = open("_site/#{local_file}")
      remote_file.content_type = local_mime ? local_mime.content_type : "text/html"
      remote_file.save
      puts " |  #{local_file} is new."
      changes = true
    end
  end

  unless changes
    puts "  (no files to upload)"
  end

  puts
  puts "Checking for remote files to delete..."
  changes = false
  remote_files.each do |remote_file|
    unless File.exists? "_site/#{remote_file.key}"
      remote_file.destroy
      puts " |  #{remote_file.key} has been removed."
      changes = true
    end
  end

  unless changes
    puts "  (no files to delete)"
  end

  puts
end
