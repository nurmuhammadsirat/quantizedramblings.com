def quit(msg)
  puts msg
  exit 1
end

def ask_for_title
  print "Title Of Post: "
  STDIN.gets.chomp
end

def create_asset_folder_for(title)
  asset_path = "assets/#{normalize_title_to_file_name(title)}"
  Dir.mkdir(asset_path)
  asset_path
end

def normalize_title_to_file_name(str)
  str.downcase.gsub(/[^0-9a-z ]/i, '').gsub(' ', '-')
end

desc "Creates a new draft post"
task :new, [:title] do |t, args|
  title = args[:title] || ask_for_title
  quit "Unable to create a draft without a title" if title.nil? || title.empty?

  draft_file = "_drafts/#{normalize_title_to_file_name(title)}.md"
  quit "A draft already exists with that title" if File.exists?(draft_file)

  timestamp = Time.now.strftime('%Y-%m-%d %H:%M')
  asset_path = create_asset_folder_for title
  File.open(draft_file, 'w') do |fh|
    fh << "---\n"
    fh << "title: #{title}\n"
    fh << "asset_path: /#{asset_path}\n"
    fh << "updated: #{timestamp}\n"
    fh << "---\n"
    fh << "\n"
  end
  puts "Created draft: #{draft_file}"
end

desc "Creates a new post without a draft"
task :new_post, [:title] do |t, args|
  title = args[:title] || ask_for_title
  quit "Unable to create a post without a title" if title.nil? || title.empty?

  datetimestamp = Time.now.strftime('%Y-%m-%d %H:%M')
  datestamp = datetimestamp.split(' ').first
  post_file = "_posts/#{datestamp}-#{normalize_title_to_file_name(title)}.md"
  quit "A post already exists with that title" if File.exists?(post_file)

  asset_path = create_asset_folder_for title
  File.open(post_file, 'w') do |fh|
    fh << "---\n"
    fh << "title: #{title}\n"
    fh << "asset_path: /#{asset_path}\n"
    fh << "updated: #{datetimestamp}\n"
    fh << "---\n"
    fh << "\n"
  end
  puts "Created post: #{post_file}"
end

desc "Converts a draft to a post"
task :convert, [:title] do |t, args|
  title = args[:title] || ask_for_title
  quit "Unable to find a draft without a title" if title.nil? || title.empty?

  draft_file = "#{normalize_title_to_file_name(title)}.md"
  quit "Unable to find a draft with that title: #{title}" unless File.exists?("_drafts/#{draft_file}")

  datetimestamp = Time.now.strftime('%Y-%m-%d %H:%M')
  datestamp = datetimestamp.split(' ').first
  post_file = "_posts/#{datestamp}-#{draft_file}"
  content = File.read("_drafts/#{draft_file}")
  content.gsub!(/^updated:.*$/, "updated: #{datetimestamp}")
  File.open(post_file, 'w') do |fh|
    fh.write content
  end
  File.unlink "_drafts/#{draft_file}"
  puts "Deleted draft: #{draft_file}"
  puts "Created post: #{post_file}"
end
