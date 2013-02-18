require 'xp5k'
# https://github.com/pmorillon/xp5k

XP5K::Config.load

@osxp = XP5K::XP.new(:logger => logger)

@osxp.define_job({
  :resources  => XP5K::Config[:resources] || "{type='kavlan'}/vlan=1+/nodes=1,walltime=4",
  :site       => XP5K::Config[:site] || 'nancy',
  :types      => ['deploy','destructive'],
  :name       => 'OpenStackXp_ctrl',
  :command    => 'sleep 86400'
})

@osxp.define_job({
  :resources  => XP5K::Config[:resources] || 'nodes=2,walltime=4',
  :site       => XP5K::Config[:site] || 'nancy',
  :types      => ['deploy','destructive'],
  :name       => 'OpenStackXp_cmpt',
  :command    => 'sleep 86400'
})


@osxp.define_deployment({
  :site             => XP5K::Config[:site] || 'nancy',
  :environment      => 'wheezy-x64-nfs',
  :jobs             => %w{ OpenStackXp_cmpt OpenStackXp_ctrl },
  :key              => File.read(XP5K::Config[:public_key]),
  :vlan             => XP5K::Config[:vlan],
  :partition_number => '3'
})

role :ctrl do
  ctrl = []
  ctrl << "#{@osxp.job_with_name('OpenStackXp_ctrl')['assigned_nodes'].first.split('.')[0]}-kavlan-#{XP5K::Config[:vlan]}.loc"
  ctrl
end

role :cmpt do
  ctrl = []
  @osxp.job_with_name('OpenStackXp_cmpt')['assigned_nodes'].each do |n|
    ctrl << "#{n.split('.')[0]}-kavlan-#{XP5K::Config[:vlan]}.loc"
  end
  ctrl
end

desc 'Submit jobs'
task :submit do
  @osxp.submit
end

desc 'Deploy with Kadeplopy'
task :deploy do
  @osxp.deploy
end

desc 'Status'
task :status do
  @osxp.status
end

desc 'Remove all running jobs'
task :clean do
  logger.debug "Clean all Grid'5000 running jobs..."
  @osxp.clean
end

desc 'Run specified command on all nodes involved'
task :cmd, :roles => [:ctrl,:cmpt] do
  set :user, 'root'
  run ENV['CMD']
end


desc 'Install puppet on hosts'
task :puppet, :roles=>[:ctrl,:cmpt] do
  set :user, 'root'
  run "export DEBIAN_FRONTEND=noninteractive;apt-get update"
  run "apt-get install -y puppet"
end
