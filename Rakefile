namespace :book do
    EXT = 'adoc'
    INFILE = "book.#{EXT}"
    OUTFILE = 'book'
    SRC = 'src'
    # Variables referenced for build
    git_tags_string = `git describe --tags`.chomp
    if git_tags_string.empty?
        git_tags_string = '0'
    end
    date_string = Time.now.strftime('%Y-%m-%d')
    # The time in milliseconds since the January 1, 1970 epoch is useful for cache busting of include URIs.
    date_time_string = Time.now.strftime('%s')
    # rouge_style='bw'
    rouge_style='vscodedark'
    params = "-r ./themes/vscodedark.rb -a rouge-style='#{rouge_style}' -a gittags='#{git_tags_string}' -a revdate='#{date_string}' -a revdatetime='#{date_time_string}' -a allow-uri-read"
    header_hash = `git rev-parse --short HEAD`.strip

    # Check contributors list
    # This checks commit hash stored in the header of list against current HEAD
    def check_contrib
        if File.exist?("#{SRC}/contributors.txt")
            current_head_hash = `git rev-parse --short HEAD`.strip
            header = `head -n 1 src/contributors.txt`.strip
            # Match regex, then coerce resulting array to string by join
            header_hash = header.scan(/[a-f0-9]{7,}/).join

            if header_hash == current_head_hash
                puts "Hash on header of contributors list (#{header_hash}) matches the current HEAD (#{current_head_hash})"
            else
                puts "Hash on header of contributors list (#{header_hash}) does not match the current HEAD (#{current_head_hash}), refreshing"
                sh "rm #{SRC}/contributors.txt"
                # Reenable and invoke task again
                Rake::Task["#{SRC}/contributors.txt"].reenable
                Rake::Task["#{SRC}/contributors.txt"].invoke
            end
        end
    end

    desc 'build basic book formats'
    task :build => [:build_html, :build_epub, :build_pdf, :build_mobi] do
        begin
            # Run check
            Rake::Task['book:check'].invoke

            # Rescue to ignore checking errors
        rescue => e
            puts e.message
            puts 'Error when checking books (ignored)'
        end
    end

    desc 'build basic book formats (for ci)'
    task :ci => [:build_html, :build_epub, :build_pdf] do
        # Run check, but don't ignore any errors
        Rake::Task['book:check'].invoke
    end

    desc 'generate contributors list'
    file "#{SRC}/contributors.txt" do
        puts 'Generating contributors list'
        sh "echo 'Contributors as of #{header_hash}:\n' > #{SRC}/contributors.txt"
        sh "git shortlog -s | grep -v -E '(Holmes|dependabot)' | cut -f 2- | column -c 120 >> #{SRC}/contributors.txt"
    end

    desc 'build Epub format'
    task :build_epub => "#{SRC}/contributors.txt" do
        check_contrib()

        puts 'Converting to EPub...'
        sh "bundle exec asciidoctor-epub3 #{params} -a imagesdir=images -a rouge-style=github #{INFILE} -o dist/#{OUTFILE}.epub"
        puts ' -- Epub output at book.epub'
    end

    desc 'build HTML format'
    task :build_html => "#{SRC}/contributors.txt" do
        check_contrib()

        puts 'Converting to HTML...'
        sh "bundle exec asciidoctor #{params} -a imagesdir=images -a data-uri #{INFILE} -o dist/#{OUTFILE}.html"
        puts ' -- HTML output at book.html'
    end

    desc 'build Mobi format'
    task :build_mobi => "#{SRC}/contributors.txt" do
        check_contrib()

        puts "Converting to Mobi (kf8)..."
        sh "bundle exec asciidoctor-epub3 #{params} -a imagesdir=images -a ebook-format=kf8 #{INFILE} -o dist/#{OUTFILE}.mobi"
        puts " -- Mobi output at book.mobi"
    end

    desc 'build PDF format'
    task :build_pdf => "#{SRC}/contributors.txt" do
        check_contrib()

        puts 'Converting to PDF... (this one takes a while)'
        sh "bundle exec asciidoctor-pdf #{params} -a imagesdir=images -a pdf-fontsdir=themes #{INFILE} -o dist/#{OUTFILE}.pdf 2>/dev/null"
        puts ' -- PDF output at book.pdf'
    end

    desc 'Check generated books'
    task :check => [:build_html, :build_epub] do
        puts 'Checking generated books'

        # sh "htmlproofer dist/#{OUTFILE}.html"
        # sh "epubcheck   dist/#{OUTFILE}.epub"
    end

    desc 'Clean all generated files'
    task :clean do
    begin
        puts 'Removing generated files'

        FileList["#{SRC}/contributors.txt", "dist/#{OUTFILE}.html", "dist/#{OUTFILE}.epub", "dist/#{OUTFILE}.mobi", "dist/#{OUTFILE}.pdf"].each do |file|
            rm file

            # Rescue if file not found
            rescue Errno::ENOENT => e
              begin
                  puts e.message
                  puts 'Error removing files (ignored)'
              end
        end
    end
  end

end

task :default => "book:build"
