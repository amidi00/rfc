task :environment do
  require 'bundler'
  Bundler.setup
  require_relative 'app'
end

namespace :db do
  task :rebuild => :environment do
    DataMapper.auto_migrate!
  end

  task :migrate => :environment do
    DataMapper.auto_upgrade!
  end
end

task :import_index => ['tmp/rfc-index.xml', :environment] do |task|
  require 'nokogiri'
  require 'active_support/core_ext/object/try'
  require 'date'

  DataMapper.logger.set_log($stderr, :warn)

  index = Nokogiri File.open(task.prerequisites.first)
  num = 0

  date_from_xml = ->(xml_date) {
    if xml_date
      year = xml_date.at('./year').text
      month_name = xml_date.at('./month').text
      day = xml_date.at('./day').try(:text)
      Date.parse [year, month_name, day].join(' ')
    end
  }

  index.search('rfc-entry').each do |xml_entry|
    doc_id = xml_entry.at('./doc-id').text
    unless entry = RfcEntry.get(doc_id)
      entry = RfcEntry.new
      entry.document_id = doc_id
      entry.title       = xml_entry.at('./title').text
      entry.abstract    = xml_entry.at('./abstract').try(:inner_html)
      entry.keywords    = xml_entry.search('./keywords/*').map(&:text)
    end
    entry.obsoleted     = xml_entry.search('./obsoleted-by').any?
    entry.publish_date  = date_from_xml.(xml_entry.at('./date'))
    num += 1 if entry.dirty?
    entry.save!
  end

  puts "updated #{num} entries."
end

file 'tmp/rfc-index.xml' do |task|
  mkdir_p 'tmp'
  index_url = 'ftp://ftp.rfc-editor.org/in-notes/rfc-index.xml'
  sh 'curl', '-#', index_url, '-o', task.name
end
