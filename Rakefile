require 'http'
require 'json'
require 'active_support'
require 'active_support/core_ext'

task :dates do
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

  File.write("questions-and-answers/dates.md", questions.map { |qa| "## #{qa[:q]}\n#{qa[:a]}" }.join("\n\n"))
end

task :ministers do
  questions = []
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

  File.write("questions-and-answers/ministers.md", questions.map { |qa| "## #{qa[:q]}\n#{qa[:a]}" }.join("\n\n"))
end

task :orgs do
  questions = []

  orgs = JSON.parse(HTTP.get(
    "https://www.gov.uk/api/search.json?filter_format=organisation&count=50" # only 50 for development speed 
  ))["results"]

  orgs.each do |org|
    details = JSON.parse(HTTP.get("https://www.gov.uk/api/content#{org["link"]}"))["details"]
    puts org["title"]

    details["social_media_links"].to_a.each do |link|
      case link["service_type"]
      when "twitter", "facebook", "youtube", "linkedin", "instagram"
        questions << {
          q: "What is the #{link["service_type"]} account for #{org["title"]}?",
          a: "The #{link["service_type"]} account for #{org["title"]} is #{link["href"]}",
        }
      when "blog"
        questions << {
          q: "What is the #{link["service_type"]} for #{org["title"]}?",
          a: "The blog for #{org["title"]} is #{link["href"]}",
        }
      end
    end
  end

  File.write("questions-and-answers/orgs.md", questions.map { |qa| "## #{qa[:q]}\n#{qa[:a]}" }.join("\n\n"))
end
