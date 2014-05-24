require 'rubygems'
require 'bundler/setup'

require 'rake/clean'

distro = nil
fpm_opts = ""

if File.exist?('/etc/system-release') && File.read('/etc/redhat-release') =~ /centos|redhat|fedora|amazon/i
  distro = 'rpm'
  fpm_opts << " --rpm-user root --rpm-group root "
elsif File.exist?('/etc/os-release') && File.read('/etc/os-release') =~ /ubuntu|debian/i
  distro = 'deb'
  fpm_opts << " --deb-user root --deb-group root "
  fpm_opts << " --depends google-chrome-stable "
end

unless distro
  $stderr.puts "Don't know what distro I'm running on -- not sure if I can build!"
end

CLEAN.include("jailed-root", 'pkg')

desc "build chromedriver #{distro}"
task :default => :clean do
  version = "unknown"
  release = ENV['GO_PIPELINE_COUNTER'] || ENV['RELEASE'] || 1
  name = 'google-chrome-driver'

  mkdir_p "jailed-root/usr/local/bin"

  mkdir_p "pkg"
  mkdir_p "downloads"

  version = nil
  cd "downloads" do
    if distro == 'rpm'
      sh("curl --fail --location http://chromedriver.googlecode.com/files/chromedriver_linux64_26.0.1383.0.zip > chromedriver_linux64.zip")
    elsif distro == 'deb'
      sh("curl --fail --location http://chromedriver.storage.googleapis.com/2.4/chromedriver_linux64.zip > chromedriver_linux64.zip")
    end
  end

  cd "jailed-root/usr/local/bin" do
    sh('unzip -q ../../../../downloads/chromedriver_linux64.zip')
    sh('mv chromedriver chromedriver-original')
    File.open('chromedriver', 'w') do |f|
      f.write(<<-BASH)
        #!/bin/bash

        chromedriver-original --no-sandbox $@
      BASH
    end
    sh('chmod 755 chromedriver-original')
    sh('chmod 755 chromedriver')
  end

  if distro == 'rpm'
    version = %x[(jailed-root/usr/local/bin/chromedriver --version & PID=$!; sleep 2; kill $PID)].match(/^version=(.*)/)[1]
  elsif distro == 'deb'
    version = %x[(jailed-root/usr/local/bin/chromedriver --version & PID=$!; sleep 2; kill $PID)].match(/\(v(.*)\)/)[1]
  end

  raise 'could not determine version' if version.nil? || version.empty?

  cd "pkg" do
    sh(%Q{
         bundle exec fpm -s dir -t #{distro} --name #{name} -a x86_64 --version "#{version}" --iteration #{release} -C ../jailed-root --verbose #{fpm_opts} --maintainer snap-ci@thoughtworks.com --vendor snap-ci@thoughtworks.com --url http://snap-ci.com --description "Heroku Toolbelt" .
    })
  end
end
