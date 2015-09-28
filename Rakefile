require 'rubygems'
require 'bundler/setup'

distro = nil
fpm_opts = ""
dependent_packages = ""

if File.exist?('/etc/system-release') && File.read('/etc/redhat-release') =~ /centos|redhat|fedora|amazon/i
  distro = 'rpm'
  fpm_opts << " --rpm-user root --rpm-group root "
elsif File.exist?('/etc/os-release') && File.read('/etc/os-release') =~ /ubuntu|debian/i
  dependent_packages = %w(libgconf-2-4)
  distro = 'deb'
  fpm_opts << " --deb-user root --deb-group root "
  fpm_opts << " --depends google-chrome-stable "
end

unless distro
  $stderr.puts "Don't know what distro I'm running on -- not sure if I can build!"
end

package_manager = distro == 'deb' ? 'apt-get' : 'yum'

versions = %w(2.10 2.11 2.12 2.13 2.14 2.15 2.16 2.17 2.18 2.19)
release = Time.now.utc.strftime('%Y%m%d%H%M%S')

task :clean do
  rm_rf "pkg"
  rm_rf "downloads"
end

namespace "wrapper" do
  task :clean do
    rm_rf "jailed-root"
  end

  task :init do
    mkdir_p  "jailed-root/usr/local/bin"
    mkdir_p "pkg"
  end

  desc "build the chromedriver wrapper package"
  task :build => [:clean, :init] do

    cd "jailed-root/usr/local/bin" do
      File.open('chromedriver', 'w') do |f|
        f.write(<<-BASH)
#!/bin/bash

DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
export LD_LIBRARY_PATH=/opt/google/chrome/lib:$LD_LIBRARY_PATH
unset RUBYOPT BUNDLE_GEMFILE RUBYLIB BUNDLE_BIN_PATH GEM_HOME GEM_PATH

# allow users to pick up a chromedriver version by specifying an environment variable
CHROMEDRIVER_VERSION=${CHROMEDRIVER_VERSION:-2.10}

exec ${DIR}/chromedriver-original-${CHROMEDRIVER_VERSION} "$@"

BASH
      end # File.open
      sh('chmod 755 chromedriver')
    end # cd

    cd "pkg" do
      sh(%Q{
         bundle exec fpm -s dir -t #{distro} --name google-chrome-driver-wrapper -a x86_64 --version "1.0" --iteration #{release} -C ../jailed-root --verbose #{fpm_opts} --maintainer support@snap-ci.com --vendor support@snap-ci.com --url http://snap-ci.com --description "Chromedriver wrapper" .
      })
    end
  end
end

versions.each do |version|
  namespace version do
    package_name = "google-chrome-driver-#{version}"

    task :clean do
      rm_rf "jailed-root"
    end

    task :init do
      mkdir_p  "jailed-root/usr/local/bin"
      mkdir_p "pkg"
      mkdir_p "downloads"
    end

    desc "build chromedriver #{version} #{distro}"
    task :build => [:clean, :init] do

      cd "downloads" do
	sh("sudo #{package_manager} install -y #{dependent_packages.join(' ')}") if dependent_packages
        sh("curl --silent --fail --location https://chromedriver.storage.googleapis.com/#{version}/chromedriver_linux64.zip > chromedriver_linux64.zip")
      end

      cd "jailed-root/usr/local/bin" do
        sh('unzip -q ../../../../downloads/chromedriver_linux64.zip')
        sh("mv chromedriver chromedriver-original-#{version}")
        sh("chmod 755 chromedriver-original-#{version}")
      end

      upstream_version = %x[(
        export LD_LIBRARY_PATH=/opt/google/chrome/lib:$LD_LIBRARY_PATH
        unset RUBYOPT BUNDLE_GEMFILE RUBYLIB BUNDLE_BIN_PATH GEM_HOME GEM_PATH
        jailed-root/usr/local/bin/chromedriver-original-#{version} --version
        )].match(%r{(#{version}\.\d+)})[1]

      raise 'could not determine version' if upstream_version.nil? || upstream_version.empty?

      cd "pkg" do
	dependent_packages_argument = dependent_packages.map {|pkg| "--depends #{pkg}" }.join(' ')
        sh(%Q{
             bundle exec fpm -s dir -t #{distro} --name #{package_name} -a x86_64 --version "#{upstream_version}" --iteration #{release} -C ../jailed-root --verbose #{fpm_opts} --depends google-chrome-driver-wrapper #{dependent_packages_argument} --maintainer support@snap-ci.com --vendor support@snap-ci.com --url http://snap-ci.com --description "Chromedriver binary" .
        })
      end
    end
  end

  task :default => [:clean, "#{version}:build", "wrapper:build"]
end
