# test



■ansibleのディレクトリ体系

サーバ追加時に追加、変更するのは基本以下のみとする。
/etc/ansible/hosts
/etc/ansible/group_vars
/etc/ansible/host_vars

/etc
  |
  |-ansible
     |
     |-group_vars　　グループ個別のパラメータを'グループ名.yml'に格納
     |   |-T1.yml
     |
     |
     |-host_vars　　　サーバ個別のパラメータを'ホスト名.yml'に格納
     |   |-SV01.yml
     |   |-SV02.yml
     |
     |-roles
     |   |-common　ディレクトリ名はロール名にする
     |   |  |-tasks
     |   |  |   |-main.yml　実行するタスクを記載
     |   |  |
     |   |  |-handlers
     |   |  |   |-main.yml　tasks/main.ymlで実行したタスク実行に実行するタスクを記載(サービス再起動など)
     |   |  |
     |   |  |-templates　　　クライアントに転送するファイル(変数は使用する）
     |   |  |   |-network
     |   |  |   |-
     |   |  |
     |   |  |-files　　　　　クライアントに転送するファイル(変数は使用しない）
     |   |  |   |-hosts
     |   |  |   |-ipv6_disable
     |   |  |   |-epel-release-6-8.noarch.rpm
     |   |  |
     |   |  |-vars　　　ロール個別に設定する変数を格納
     |   |      |-main.yml
     |   |
     |   |-web
     |      |-tasks
     |      |   |-main.yml
     |      |
     |      |-handlers
     |      |   |-main.yml
     |      |
     |      |-templates
     |      |   |-network
     |      |   |-
     |      |
     |      |-files
     |      |   |-hosts
     |      |   |-ipv6_disable
     |      |   |-epel-release-6-8.noarch.rpm
     |      |
     |      |-vars　　　ロール個別に設定する変数を格納
     |          |-main.yml
     |
     |
     |-hosts　　処理対象サーバ、グループを記載
     |-site.yml  実行する処理(実際にはロール)を記載


■serverspecのディレクトリ体系

ホスト別にテストコードを作成する。
common_spec.rbなどの中で、ansibleのホスト個別の変数が格納された
ファイル(/etc/ansible/host_vars/SV01.ymlなど)を読み込み、テストを実行する。
※/home/serverspec/SV01をSV02の下にあるテストコードの中身は同じ。


/home
  |-serverspec
     |
     |-SV01　　　　　ホスト別にテストコードを分ける。
     |   |-Rakefile
     |   |-spec
     |      |-spec_helper.rb
     |      |-SV01　　　　　　テストコードを格納。ここのディレクトリ名がテスト実行先ホストになる。(IPアドレスでもよい）
     |          |-common_spec.rb
     |          |-network_spec.rb
     |          |-web_spec.rb
     |          |-ap_spec.rb
     |
     |-SV02
     |   |-Rakefile
     |   |-spec
     |      |-spec_helper.rb
     |      |-SV02
     |          |-common_spec.rb
     |          |-network_spec.rb
     |          |-web_spec.rb
     |          |-ap_spec.rb
     |

##########################################################################
# cat sample_spec.rb
require 'spec_helper'
require 'yaml'

#properties = YAML.load_file('/etc/ansible/host_vars/ansible02.yml')
properties = YAML.load_file("/etc/ansible/host_vars/" + ENV['TARGET_HOST'] + ".yml")

describe package(properties['pkg1']), :if => os[:family] == 'redhat' do
  it { should be_installed }
end

describe package('properties.pkg1'), :if => os[:family] == 'redhat' do
  it { should be_installed }
end

describe package('apache2'), :if => os[:family] == 'ubuntu' do
  it { should be_installed }
end

describe service('httpd'), :if => os[:family] == 'redhat' do
  it { should be_enabled }
  it { should be_running }
end

describe service('apache2'), :if => os[:family] == 'ubuntu' do
  it { should be_enabled }
  it { should be_running }
end

describe service('org.apache.httpd'), :if => os[:family] == 'darwin' do
  it { should be_enabled }
  it { should be_running }
end

describe port(80) do
  it { should be_listening }
end

#File.open('/etc/hosts') do |file|
#HOSTS_MD5 = `cksum /etc/hosts|awk '{print $1}'`
HOSTS = `sed ':loop; N; $!b loop; ;s/\\n/,/g' /etc/hosts`
#  describe command("cksum /etc/hosts|awk '{print $1}'") do
  describe command("sed ':loop; N; $!b loop; ;s/\\n/,/g' /etc/hosts") do
#  describe command("cat /etc/hosts") do
#    its(:stdout) { should match HOSTS_MD5 }
    its(:stdout) { should match HOSTS }
  end
#end

###########################################################################
# cat ansible02.yml

hosts: ansible02
bond: yes
pkg1: wget


###########################################################################

# more main.yml

 - name: pkg install
   yum: name={{ item }} state=present
   with_items:
   - libselinux-python
   - wget

