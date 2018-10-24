require 'http'
require 'json'
require 'yaml'
require 'active_support'
require 'active_support/core_ext'

task :steps do
  questions = []
  step_by_steps = JSON.parse(HTTP.get("https://www.gov.uk/api/search.json?filter_content_store_document_type=step_by_step_nav"))["results"]
  step_by_steps.each do |s|
    title = s["title"].gsub(": step by step", "").gsub("you've", "I've")

    item =  JSON.parse(HTTP.get("https://www.gov.uk/api/content#{s["link"]}"))["details"]

    a = "This is a #{item["step_by_step_nav"]["steps"].size} step process. The first thing to do is to #{item["step_by_step_nav"]["steps"][0]["title"]}. Search for '#{title}' on GOV.UK to continue."

    questions << {
      q: "How do I #{title}?",
      a: a,
    }
  end

  write("steps.md", questions)
end

task :clocks do
  questions = []
  changes = JSON.parse(HTTP.get("https://www.gov.uk/when-do-the-clocks-change.json"))["united-kingdom"]["events"]

  next_change = changes.sort_by { |change| Date.parse(change["date"]) }.select { |change| Date.parse(change["date"]) > Date.today }.first

  questions << {
    q: "When do the clocks change?",
    a: "The #{next_change["notes"]} on #{Date.parse(next_change["date"]).to_formatted_s(:long)}",
  }

  write("clocks.md", questions)
end

task :cma do
  questions = []
  cma_cases = JSON.parse(HTTP.get("https://www.gov.uk/api/search.json?filter_content_store_document_type=cma_case&fields[]=title,case_state&count=1000"))["results"]

  cma_cases.each do |cma_case|
    questions << {
      q: "What's the status of the #{cma_case["title"]} CMA case?",
      a: "It is #{cma_case["case_state"].first["label"]}",
    }
  end

  write("cma.md", questions)
end

def write(file, questions)
  puts YAML.dump(questions)
  File.write("questions-and-answers/#{file}", questions.map { |qa| "## #{qa[:q]}\n#{qa[:a]}" }.join("\n\n") + "\n")
end

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
