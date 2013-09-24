require 'rake'

task :default => 'install:full_install'

# Main install bootstrap
namespace :install do
	desc "Install dotfiles"
	task :full_install do
		puts ""
		Rake::Task['git:ensure_installed'].invoke
		Rake::Task['zsh:ensure_installed'].invoke
		Rake::Task['zsh:ensure_oh_my_zsh'].invoke
		Rake::Task['zsh:install_oh_my_zsh_syntax_highlighting'].invoke
		Rake::Task['dotfiles:install'].invoke
	    puts ""
	    puts "========================================".green
	    puts ""
	    puts "Install completed!".green
	    puts ""
	    puts "You should probably `source ~/.zshrc` to load everything up.".green
	    puts ""
	    puts "Additionally, if you changed your shell to ZSH from something else, you should log out and log in again.".green
	    puts ""
	    puts "========================================".green
	    puts ""
	end
end

# Tasks related to git
namespace :git do
	desc "Ensure git is installed"
	task :ensure_installed do
		puts "Checking to see if git is installed"
		exists = system "which -s git"
		if exists
			puts "    Git is installed!".green
			puts ""
		else
			puts "    Git does not appear to be available. Please install git before proceeding. Aborting".bold.white.bg_red
			puts ""
			exit 0
		end
	end
end

# Tasks related to ZSH
namespace :zsh do

	def oh_my_zsh_installed;	system "ls -al ~ | grep .oh-my-zsh > /dev/null" end

	desc "Ensure ZSH is installed"
	task :ensure_installed do
		puts "Checking to see if ZSH is installed"
		exists = system "which -s zsh"
		if exists
			puts "    ZSH is installed!".green
			puts ""
			Rake::Task['zsh:ensure_available'].invoke
		else
			puts "    ZSH does not appear to be installed. Please install ZSH before proceeding. Aborting".bold.white.bg_red
			puts ""
			exit 0
		end
	end

	desc "Ensure ZSH is listed as an available shell"
	task :ensure_available do
		puts "Checking to see if zsh is listed in our available shells"
		available = system "cat /etc/shells | grep zsh > /dev/null"
		if available
			puts "    ZSH is available!".green
			puts ""
			Rake::Task['zsh:ensure_set'].invoke
		else
			puts "    ZSH does not appear to be available. Please ensure ZSH is an available shell before proceeding. Aborting".bold.white.bg_red
			puts ""
			exit 0
		end
	end

	desc "Ensure ZSH is set as the default shell"
	task :ensure_set do
		puts "Making sure ZSH is set to our default shell"
		default = system "echo $SHELL | grep zsh > /dev/null"
		if default
			puts "    ZSH is set as our default shell!".green
			puts ""
		else
			puts "    ZSH does not appear to be set as our default shell. Attempting to set as default".red
			puts ""
			Rake::Task['zsh:set_zsh'].invoke
		end
	end

	desc "Set ZSH as the default shell"
	task :set_zsh do
		puts "Setting ZSH as our default shell"
		zsh_path = `cat /etc/shells | grep zsh`
		system "chsh -s " + "#{zsh_path}"
		puts "    Set ZSH as default shell!".green
		puts ""
	end

	desc "Ensure Oh My ZSH is available"
	task :ensure_oh_my_zsh do
		puts "Checking to see if Oh My ZSH is installed"
		installed = oh_my_zsh_installed()
		if installed
			puts "    Oh My ZSH is installed!".green
			puts ""
		else
			puts "    Oh My ZSH does not appear to be installed. Attempting to install".red
			puts ""
			Rake::Task['zsh:install_oh_my_zsh'].invoke
		end
	end

	desc "Install Oh My ZSH"
	task :install_oh_my_zsh do
		puts "Attempting to install Oh My ZSH"
		system "git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh"
		installed = oh_my_zsh_installed()
		if installed
			puts "    Installed Oh My ZSH!".green
			puts ""
		else
			puts "    Error attempting to install Oh My ZSH. Aborting".bold.white.bg_red
			puts ""
		end
	end

	desc "Install Oh My ZSH syntax highlighting plugin"
	task :install_oh_my_zsh_syntax_highlighting do
		puts "Checking to see if zsh-syntax-highlighting plugin for Oh My ZSH is installed"
		if File.exist? File.join(ENV['HOME'], "/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.plugin.zsh")
			puts "    zsh-syntax-highlighting plugin already installed!".green
			puts ""
		else
			puts "    Attempting to install zsh-syntax-highlighting plugin for Oh My ZSH."
			puts ""
			system "git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $HOME/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting"
			puts ""
		end
	end
end

# Tasks related to dotfiles
namespace :dotfiles do
	desc "Install dotfiles in user's home directory"
	task :install do
		puts "Installing dotfiles"
		puts ""
		replace_all = false
		Dir['*'].each do |file|
		    next if %w[Rakefile README.md LICENSE].include? file

		    if File.exist? File.join(ENV['HOME'], ".#{file}")
				if File.identical? file, File.join(ENV['HOME'], ".#{file}")
					puts "Identical #{file} already installed."
				elsif replace_all
					replace_file(file)
				else
					print "File already exists: ~/.#{file}. Overwrite it? [y,n,q,a,?] "
				case $stdin.gets.chomp
				when 'y'
					replace_file(file)
				when 'a'
					replace_all = true
					replace_file(file)
				when 'q'
					exit
				when '?'
					puts "y - Overwrite the file"
					puts "n - Don't overwrite the file"
					puts "q - Don't overwrite this nor all subsequent other files"
					puts "a - Overwrite this and all subsequent other files"
					puts "? - Print help"
					redo
				end
				end
		    else link_file(file)
		    end
		end
	end
end

# Replace the given dotfile in $HOME with the target file.
def replace_file(file)
	system %Q{rm -rf "$HOME/.#{file}"}
	link_file(file)
end

# Create a symlink to the target file in $HOME.
def link_file(file)
    puts "Linking #{file} to home directory."
    system %Q{ln -s "$PWD/#{file}" "$HOME/.#{file}"}
end

# Some fancy colors
class String
	def bold;           "\033[1m#{self}\033[22m" end
	def white;          "\033[37m#{self}\033[0m" end
	def red;            "\033[31m#{self}\033[0m" end
	def bg_red;         "\033[41m#{self}\033[0m" end
	def green;          "\033[32m#{self}\033[0m" end
end