require "rake"
require "rake/testtask"
require "rake/clean"

task :default => [:test]

task :build => :clean do
  yaml = File.open("src/compress.yaml").read
  liquid = File.open("src/compress.liquid").read.gsub(/\s+(?={)/, "").gsub(/{% comment %}[^{]+{% endcomment %}/, "")

  mkdir_p "_build"
  File.open "_build/compress.html", File::CREAT|File::WRONLY do |f|
    f.puts yaml, "", liquid
  end
end

Rake::TestTask.new :test => :build do |t|
  t.libs << "test"
  t.test_files = FileList["test/test_*.rb"]
end

task :performance => :build do
  require "benchmark"
  Dir.chdir "performance" do
    puts Benchmark.measure { sh "bundle exec jekyll build" }
  end
end

CLEAN.include FileList["_build/compress.html"]

namespace :site do
  task :build do
    Dir.chdir "site" do
      sh "bundle exec jekyll build"
    end
  end

  task :test => :build do
    Dir.chdir "site" do
      sh "wget -O vnu.zip https://github.com/validator/validator/releases/download/20141006/vnu-20141013.jar.zip"
      sh "unzip vnu.zip"
      sh "java -jar ./vnu/vnu.jar ./_site/"
    end
  end

  desc "Commit the local site to the gh-pages branch and publish to GitHub Pages"
  task :publish do
    GH_PAGES_DIR = "_gh-pages"

    # Ensure the gh-pages dir exists so we can generate into it.
    puts "Checking for gh-pages dir..."
    unless File.exist? GH_PAGES_DIR
      puts "Creating gh-pages dir..."
      sh "git clone git@github.com:penibelst/jekyll-compress-html #{GH_PAGES_DIR}"
    end

    # Ensure latest gh-pages branch history.
    Dir.chdir GH_PAGES_DIR do
      sh "git checkout gh-pages"
      sh "git pull origin gh-pages"
    end

    puts "Cleaning gh-pages directory..."
    rm_rf FileList[File.join(GH_PAGES_DIR, "**", "*")], { :verbose => true }

    puts "Copying site to gh-pages branch..."
    cp_r FileList[File.join("site", "*")].include(".gitignore"), GH_PAGES_DIR

    puts "Committing and pushing to GitHub Pages..."
    sha = `git log`.match(/[a-z0-9]{40}/)[0]
    Dir.chdir GH_PAGES_DIR do
      sh "git add ."
      sh "git commit --allow-empty -m 'Updating to #{sha}.'"
      sh "git push origin gh-pages"
    end
    puts 'Done.'
  end
end
