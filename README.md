How to implement Event Calendar in Rails App

Event calendar is a way to show multiple, overlapping events across calendar days and rows. This is an interface to add events, edit events, & destroy event. In Rails there is a gem/plugin “event_calendar” to implement it just like google calendar.

The following steps demonstrate the implementation of event_calendar in both Rails 2.3.x and Rails3.x environment.

Step# 1– Installation of the gem/plugin
In rails 2.3.x
Install the required plugin from below path

script/plugin install git://github.com/elevation/event_calendar.git
Generate the necessary static file and example
script/generate event_calendar
In rails 3.x
Install the required gems

gem ‘event-calendar’, :require => ‘event_calendar’
Run “bundle install”
You can also use as a Plugin, to install plugin
rails plugin install git://github.com/elevation/event_calendar.git
Generate the necessary static file for the event calendar
rails generate event_calendar
Step# 2

Include the necessary style sheet & java-script into your layout/view

Step# 3

Create a migration file to add necessary columns as follows

 class CreateEvents < ActiveRecord::Migration

       def self.up

          create_table :events do |t|

                t.string :name

                t.datetime :start_at

                t.datetime :end_at

                t.timestamps

          end

       end

       def self.down

            drop_table :events 

       end

   end
Step# 4

Add the necessary paths to the “config/routes” file

In Rails 2.3.x

 
  map.calendar '/calendar/:year/:month', :controller => 'calendar', :action => 'index',

        :requirements => {:year => /\d{4}/, :month => /\d{1,2}/}, :year => nil, :month => nil
In Rails3.x

match '/calendar(/:year(/:month))' => 'calendar#index', :as => :calendar, :constraints => {:year => /\d{4}/, :month => /\d{1,2}/}
Step# 5
Change the Event model to add the calendar as follows

class Event < ActiveRecord::Base

     has_event_calendar

  end
Step# 6

Modify the Calendar controller as follows

class CalendarController < ApplicationController

     def index

       @month = (params[:month] || Time.zone.now.month).to_i

        @year = (params[:year] || Time.zone.now.year).to_i

        @shown_month = Date.civil(@year, @month)

        @event_strips = Event.event_strips_for_month(@shown_month)

     end
  end
Step# 7

You can also override the events method in helpers/calendar_helper.rb

module CalendarHelper

  	def month_link(month_date)

    	 link_to(I18n.localize(month_date, :format => "%B"), {:month => month_date.month, :year => month_date.year})

	end

  # custom options for this calendar

 	def event_calendar_options

   	 { 

      	      :year => @year,

              :month => @month,

              :event_strips => @event_strips,

              :month_name_text => I18n.localize(@shown_month, :format => "%B %Y"),

              :previous_month_text => "<< " + month_link(@shown_month.prev_month),

              :next_month_text => month_link(@shown_month.next_month) + " >>"

           }

	end

	def event_calendar

	     calendar event_calendar_options do |args|

               event = args[:event]

              %(#{h(event.name)})

	end

       end

  end
Step# 8

Add the following code to display the calendar in the view file
