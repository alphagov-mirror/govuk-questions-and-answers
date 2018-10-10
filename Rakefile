require 'http'
require 'json'
require 'active_support'
require 'active_support/core_ext'

task :generate do
  questions = []

  events = JSON.parse(HTTP.get("https://www.gov.uk/bank-holidays.json"))["england-and-wales"]["events"]

  events.each do |event|
    date = Date.parse(event["date"])

    if date > Date.today
      questions << {
        q: "When is #{event["title"]} in #{date.year}?",
        a: "The #{event["title"]} in #{date.year} is on #{event["date"]}",
      }
    end

    if date < Date.today
      questions << {
        q: "When was #{event["title"]} in #{date.year}?",
        a: "The #{event["title"]} in #{date.year} was on #{event["date"]}",
      }
    end

    if date > Date.today && date < Date.today + 1.year
      questions << {
        q: "When is the next #{event["title"]}?",
        a: "The next #{event["title"]} is on #{event["date"]}",
      }
    end
  end

  ministers = JSON.parse(HTTP.get("https://www.gov.uk/api/search.json?filter_format=minister&count=1000"))["results"]

  ministers.each do |minister|
    person = minister["description"].to_s.gsub('Current role holder:', '').strip
    if person.blank?
      puts "Could not find the person for #{minister["title"]}"
      next
    end

    questions << {
      q: "Who is #{minister["title"]}?",
      a: "The #{minister["title"]} is #{person}",
    }
  end

  File.write("questions-and-answers.md", questions.map { |qa| "## #{qa[:q]}\n#{qa[:a]}" }.join("\n\n"))
end
