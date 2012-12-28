---
layout: post
title: "Extend Twitter Bootstrap with your own jQuery plugin"
description: "A short tutorial on writing your first jQuery plugin to extend Twitter Bootstrap."
category: "Twitter Bootstrap"
tags: ["Twitter Bootstrap", "jQuery Plugins"]
---
{% include JB/setup %}

You want to extend Twitter Bootstrap with your own jQuery plugin but you don't know where
to start? This short tutorial will go over the different steps to create your first jQuery
plugin to extend Twitter Bootstrap.


In this tutorial, we will build a basic date picker jQuery plugin. The plugin will be
built in 4 steps:
1. Plugin Definition
1. Class Definition
1. Events Definition
1. Plugin Auto-Loading


### Plugin Definition

This is where you define the plugin itself. It will allow you to instantiate the plugin
on any element by calling the constructor function on the element. Using the data API, you
can store a reference to the plugin on the element. You can also define some functions
to be called directly by detecting option of type string. Finally, you can define default
values for your plugin.

	$.fn.datepicker = function (option) {
		return this.each(function () {
			var $this = $(this)
				, data = $this.data('datepicker')
				, options = typeof option == 'object' && option
			
			if (!data) $this.data('datepicker', (data = new DatePicker(this, options)))
			if (typeof option == 'string') data[option]()
		})
	}
	
	$.fn.datepicker.Constructor = DatePicker
	
	$.fn.datepicker.defaults = {
		format: "m/d/y",
		date: ""
	}


### Class Definition

This is where you define the plugin class. You define here the class prototype including
all the functions that this class will provide as well as the constructor. In our case,
the class prototype as several functions, a constructor function that takes an element
along with some options as well as a toggle selector. Our constructor also extends the
plugin default values and call an init function.

	var toggle = '[data-toggle=datepicker]'
		, DatePicker = function (element, options) {
			this.options = $.extend({}, $.fn.datepicker.defaults, options)
			this.$element = $(element)
			this.initCalendar()
		}
	
	DatePicker.prototype = {
	
		constructor: DatePicker
	
		, daysInMonth: function(month, year) {
			return new Date(year, month, 0).getDate()
		}
	  
		, dayOfWeek: function(month, year, day) {
			return new Date(year, month, day).getDay()
		}
	  
		, formatDate: function(month, year, day) {
			var date = this.options.format
			month += 1
			month = new String(month)
			day = new String(day)
			
			if (month.length == 1) {
				month = "0" + month
			}
			if (day.length == 1) {
				day = "0" + day
			}
			date = date.replace("m", month)
			date = date.replace("y", year)
			date = date.replace("d", day)
		
			return date
		}
	  
		, parse: function(element, date) {
			var format = this.options.format
			
			var monthPos = format.indexOf("m")
			var yearPos = format.indexOf("y")
			var dayPos = format.indexOf("d")
			
			var indexes = [
				{"type": "m", "pos": monthPos},
				{"type": "y", "pos": yearPos},
				{"type": "d", "pos": dayPos}
			]
		
			indexes.sort(function(a, b) {return a.pos - b.pos})
			
			var parts = date.match(/(\d+)/g)
			
			for (var i=0; i < indexes.length; i++) {
				if (indexes[i]['type'] == element) {
					return new Number(parts[i]).toString()
				}
			}
		}
	  
		, initCalendar: function() {
			var date = this.options.date
			
			if (date == "") {
				var today = new Date()
				
				this.$element.data('month', today.getMonth())
				this.$element.data('year', today.getFullYear())
			} else {
				this.$element.find('input[type=text]').val(date)
				this.$element.data('month', this.parse("m", date) - 1)
				this.$element.data('year', this.parse("y", date))
			}
			
			this.updateCalendar()
		}

		, updateCalendar: function () {
			var $calendar
				, today
				, month
				, year
				, $daysHeader
				, $days
			
			var today = new Date()
			month = this.$element.data('month')
			year = this.$element.data('year')
			
			$calendar = this.$element.find('.datepicker-calendar')
			
			$calendar.find('table > thead > tr > th.month > span').text(MonthsList[month])
			$calendar.find('table > thead > tr > th.year > span').text(year)
			$daysHeader = $calendar.find('table > thead > tr.days-header')
			$daysHeader.html('')
			for (var i=0; i < DaysList.length; i++) {
			$daysHeader.append('<th>' + DaysList[i] + '</th>')
			}
			$days = $calendar.find('table > tbody')
			$days.html('')
			var numDaysPrevious = this.daysInMonth(month, year)
			var numDays = this.daysInMonth(month + 1, year)
			var firstDay = this.dayOfWeek(month, year, 1)
			var lastDay = this.dayOfWeek(month, year, numDays)
			var row = ''
			for (var i=0; i < firstDay; i++) {
				if (i == 0) {
					row += '<tr>'
				}
				row += '<td class="off">' + (numDays - firstDay + i) + '</td>'
				if (i == 6) {
					row += '</tr>'
					$days.append(row)
					row = ''
				}
			}
			for (var i=1; i <= numDays; i++) {
				var day = this.dayOfWeek(month, year, i)
				if (day == 0) {
					row += '<tr>'
				}
				if (i == today.getDate() && month == today.getMonth() && year == today.getFullYear()) {
					row += '<td data-day="' + i + '" class="today">' + i + '</td>'
				} else {
					row += '<td data-day="' + i + '">' + i + '</td>'
				}
				if (day == 6) {
					row += '</tr>'
					$days.append(row)
					row = ''
				}
			}
			for (var i=lastDay+1, j=1; i <= 6; i++, j++) {
				if (i == 0) {
					row += '<tr>'
				}
				row += '<td class="off">' + j + '</td>'
				if (i == 6) {
					row += '</tr>'
					$days.append(row)
					row = ''
				}
			}
		}

		, previousMonth: function (e) {
			var $this = $(this)
				, $parent
				, $datePicker
			
			$parent = $this.closest('.datepicker')
			
			if ($parent.data('month') == 0) {
				$parent.data('month', 11)
				$parent.data('year', new Number($parent.data('year')) - 1)
			} else {
				$parent.data('month', new Number($parent.data('month')) - 1)
			}
			
			$datePicker = $parent.data('datepicker')
			$datePicker.updateCalendar()
			
			return false;
		}

		, nextMonth: function (e) {
			var $this = $(this)
				, $parent
				, $datePicker
			
			$parent = $this.closest('.datepicker')
			
			if ($parent.data('month') == 11) {
				$parent.data('month', 0)
				$parent.data('year', new Number($parent.data('year')) + 1)
			} else {
				$parent.data('month', new Number($parent.data('month')) + 1)
			}
			
			$datePicker = $parent.data('datepicker')
			$datePicker.updateCalendar()
			
			return false;
		}

		, previousYear: function (e) {
			var $this = $(this)
				, $parent
				, $datePicker
			
			$parent = $this.closest('.datepicker')
			
			$parent.data('year', new Number($parent.data('year')) - 1)
			
			$datePicker = $parent.data('datepicker')
			$datePicker.updateCalendar()
			
			return false;
		}

		, nextYear: function (e) {
			var $this = $(this)
				, $parent
				, $datePicker
			
			$parent = $this.closest('.datepicker')
			
			$parent.data('year', new Number($parent.data('year')) + 1)
			
			$datePicker = $parent.data('datepicker')
			$datePicker.updateCalendar()
			
			return false;
		}

		, select: function (e) {
			var $this = $(this)
				, $parent
				, $datePicker
			
			$parent = $this.closest('.datepicker')
			
			var month = $parent.data('month')
			var year = $parent.data('year')
			var day = $this.data('day')
			
			$datePicker = $parent.data('datepicker')
			
			$parent.find('input[type=text]').val($datePicker.formatDate(month, year, day))
			
			clearMenus()
			
			return false;
		}

		, toggle: function (e) {
			var $this = $(this)
				, $parent
				, isActive
			
			if ($this.is('.disabled, :disabled')) return
			
			$parent = getParent($this)
			
			isActive = $parent.hasClass('open')
			
			clearMenus()
			
			if (!isActive) {
				$parent.toggleClass('open')
			}
			
			return false
		}
	}

	function clearMenus() {
		getParent($(toggle))
		.removeClass('open')
	}

	function getParent($this) {
		var selector = $this.attr('data-target')
			, $parent
		
		if (!selector) {
			selector = $this.attr('href')
			selector = selector && /#/.test(selector) && selector.replace(/.*(?=#[^\s]*$)/, '') //strip for ie7
		}
		
		$parent = $(selector)
		$parent.length || ($parent = $this.parent())
		
		return $parent
	}


### Events Definition

This is where you define the events that will act on your plugin. For this, you can use
the on() function provided by jQuery and specialize your event by specifying a selector.
Finally, you don't need to pass a reference to the plugin as it can be accessed through
the data API.

	$(function () {
		$('html')
			.on('click.datepicker.data-api', clearMenus)
		$('body')
			.on('click.datepicker.data-api touchstart.datepicker.data-api', toggle, DatePicker.prototype.toggle)
			.on('click.datepicker.data-api touchstart.datepicker.data-api', 'table.calendar .month > .previous', DatePicker.prototype.previousMonth)
			.on('click.datepicker.data-api touchstart.datepicker.data-api', 'table.calendar .month > .next', DatePicker.prototype.nextMonth)
			.on('click.datepicker.data-api touchstart.datepicker.data-api', 'table.calendar .year > .previous', DatePicker.prototype.previousYear)
			.on('click.datepicker.data-api touchstart.datepicker.data-api', 'table.calendar .year > .next', DatePicker.prototype.nextYear)
			.on('click.datepicker.data-api touchstart.datepicker.data-api', 'table.calendar td:not[.off]', DatePicker.prototype.select)
	})
  

### Plugin Auto-Loading

To auto-load the plugin on initial rendering of the page, all you need to do is add a each
statement on the elements you want your plugin to auto-load on. You can also use the data
API to pass some values defined on the element itself. In our case, it's the class
datepicker on a div element.

	$(window).on('load', function () {
		$('div.datepicker').each(function () {
			var $datepicker = $(this)
			$datepicker.datepicker($datepicker.data())
		})
	})


### The full example

	!function ($) {
	
		"use strict"; // jshint ;_;
	
		/* DATEPICKER CLASS DEFINITION
		 * ========================= */
	  
		var toggle = '[data-toggle=datepicker]'
			, DatePicker = function (element, options) {
				this.options = $.extend({}, $.fn.datepicker.defaults, options)
				this.$element = $(element)
				this.initCalendar()
			}
		
		DatePicker.prototype = {
		
			constructor: DatePicker
		
			, daysInMonth: function(month, year) {
				return new Date(year, month, 0).getDate()
			}
		  
			, dayOfWeek: function(month, year, day) {
				return new Date(year, month, day).getDay()
			}
		  
			, formatDate: function(month, year, day) {
				var date = this.options.format
				month += 1
				month = new String(month)
				day = new String(day)
				
				if (month.length == 1) {
					month = "0" + month
				}
				if (day.length == 1) {
					day = "0" + day
				}
				date = date.replace("m", month)
				date = date.replace("y", year)
				date = date.replace("d", day)
			
				return date
			}
		  
			, parse: function(element, date) {
				var format = this.options.format
				
				var monthPos = format.indexOf("m")
				var yearPos = format.indexOf("y")
				var dayPos = format.indexOf("d")
				
				var indexes = [
					{"type": "m", "pos": monthPos},
					{"type": "y", "pos": yearPos},
					{"type": "d", "pos": dayPos}
				]
			
				indexes.sort(function(a, b) {return a.pos - b.pos})
				
				var parts = date.match(/(\d+)/g)
				
				for (var i=0; i < indexes.length; i++) {
					if (indexes[i]['type'] == element) {
						return new Number(parts[i]).toString()
					}
				}
			}
		  
			, initCalendar: function() {
				var date = this.options.date
				
				if (date == "") {
					var today = new Date()
					
					this.$element.data('month', today.getMonth())
					this.$element.data('year', today.getFullYear())
				} else {
					this.$element.find('input[type=text]').val(date)
					this.$element.data('month', this.parse("m", date) - 1)
					this.$element.data('year', this.parse("y", date))
				}
				
				this.updateCalendar()
			}
	
			, updateCalendar: function () {
				var $calendar
					, today
					, month
					, year
					, $daysHeader
					, $days
				
				var today = new Date()
				month = this.$element.data('month')
				year = this.$element.data('year')
				
				$calendar = this.$element.find('.datepicker-calendar')
				
				$calendar.find('table > thead > tr > th.month > span').text(MonthsList[month])
				$calendar.find('table > thead > tr > th.year > span').text(year)
				$daysHeader = $calendar.find('table > thead > tr.days-header')
				$daysHeader.html('')
				for (var i=0; i < DaysList.length; i++) {
				$daysHeader.append('<th>' + DaysList[i] + '</th>')
				}
				$days = $calendar.find('table > tbody')
				$days.html('')
				var numDaysPrevious = this.daysInMonth(month, year)
				var numDays = this.daysInMonth(month + 1, year)
				var firstDay = this.dayOfWeek(month, year, 1)
				var lastDay = this.dayOfWeek(month, year, numDays)
				var row = ''
				for (var i=0; i < firstDay; i++) {
					if (i == 0) {
						row += '<tr>'
					}
					row += '<td class="off">' + (numDays - firstDay + i) + '</td>'
					if (i == 6) {
						row += '</tr>'
						$days.append(row)
						row = ''
					}
				}
				for (var i=1; i <= numDays; i++) {
					var day = this.dayOfWeek(month, year, i)
					if (day == 0) {
						row += '<tr>'
					}
					if (i == today.getDate() && month == today.getMonth() && year == today.getFullYear()) {
						row += '<td data-day="' + i + '" class="today">' + i + '</td>'
					} else {
						row += '<td data-day="' + i + '">' + i + '</td>'
					}
					if (day == 6) {
						row += '</tr>'
						$days.append(row)
						row = ''
					}
				}
				for (var i=lastDay+1, j=1; i <= 6; i++, j++) {
					if (i == 0) {
						row += '<tr>'
					}
					row += '<td class="off">' + j + '</td>'
					if (i == 6) {
						row += '</tr>'
						$days.append(row)
						row = ''
					}
				}
			}
	
			, previousMonth: function (e) {
				var $this = $(this)
					, $parent
					, $datePicker
				
				$parent = $this.closest('.datepicker')
				
				if ($parent.data('month') == 0) {
					$parent.data('month', 11)
					$parent.data('year', new Number($parent.data('year')) - 1)
				} else {
					$parent.data('month', new Number($parent.data('month')) - 1)
				}
				
				$datePicker = $parent.data('datepicker')
				$datePicker.updateCalendar()
				
				return false;
			}
	
			, nextMonth: function (e) {
				var $this = $(this)
					, $parent
					, $datePicker
				
				$parent = $this.closest('.datepicker')
				
				if ($parent.data('month') == 11) {
					$parent.data('month', 0)
					$parent.data('year', new Number($parent.data('year')) + 1)
				} else {
					$parent.data('month', new Number($parent.data('month')) + 1)
				}
				
				$datePicker = $parent.data('datepicker')
				$datePicker.updateCalendar()
				
				return false;
			}
	
			, previousYear: function (e) {
				var $this = $(this)
					, $parent
					, $datePicker
				
				$parent = $this.closest('.datepicker')
				
				$parent.data('year', new Number($parent.data('year')) - 1)
				
				$datePicker = $parent.data('datepicker')
				$datePicker.updateCalendar()
				
				return false;
			}
	
			, nextYear: function (e) {
				var $this = $(this)
					, $parent
					, $datePicker
				
				$parent = $this.closest('.datepicker')
				
				$parent.data('year', new Number($parent.data('year')) + 1)
				
				$datePicker = $parent.data('datepicker')
				$datePicker.updateCalendar()
				
				return false;
			}
	
			, select: function (e) {
				var $this = $(this)
					, $parent
					, $datePicker
				
				$parent = $this.closest('.datepicker')
				
				var month = $parent.data('month')
				var year = $parent.data('year')
				var day = $this.data('day')
				
				$datePicker = $parent.data('datepicker')
				
				$parent.find('input[type=text]').val($datePicker.formatDate(month, year, day))
				
				clearMenus()
				
				return false;
			}
	
			, toggle: function (e) {
				var $this = $(this)
					, $parent
					, isActive
				
				if ($this.is('.disabled, :disabled')) return
				
				$parent = getParent($this)
				
				isActive = $parent.hasClass('open')
				
				clearMenus()
				
				if (!isActive) {
					$parent.toggleClass('open')
				}
				
				return false
			}
		}
	
		function clearMenus() {
			getParent($(toggle))
			.removeClass('open')
		}
	
		function getParent($this) {
			var selector = $this.attr('data-target')
				, $parent
			
			if (!selector) {
				selector = $this.attr('href')
				selector = selector && /#/.test(selector) && selector.replace(/.*(?=#[^\s]*$)/, '') //strip for ie7
			}
			
			$parent = $(selector)
			$parent.length || ($parent = $this.parent())
			
			return $parent
		}
	
		/* DATEPICKER PLUGIN DEFINITION
		 * ========================== */
	   
		$.fn.datepicker = function (option) {
			return this.each(function () {
				var $this = $(this)
					, data = $this.data('datepicker')
					, options = typeof option == 'object' && option
				
				if (!data) $this.data('datepicker', (data = new DatePicker(this, options)))
				if (typeof option == 'string') data[option]()
			})
		}
		
		$.fn.datepicker.Constructor = DatePicker
		
		$.fn.datepicker.defaults = {
			format: "m/d/y",
			date: ""
		}
	  
		/* APPLY TO STANDARD DATEPICKER ELEMENTS
		 * =================================== */
	
		$(window).on('load', function () {
			$('div.datepicker').each(function () {
				var $datepicker = $(this)
				$datepicker.datepicker($datepicker.data())
			})
		})
	  
		$(function () {
			$('html')
				.on('click.datepicker.data-api', clearMenus)
			$('body')
				.on('click.datepicker.data-api touchstart.datepicker.data-api', toggle, DatePicker.prototype.toggle)
				.on('click.datepicker.data-api touchstart.datepicker.data-api', 'table.calendar .month > .previous', DatePicker.prototype.previousMonth)
				.on('click.datepicker.data-api touchstart.datepicker.data-api', 'table.calendar .month > .next', DatePicker.prototype.nextMonth)
				.on('click.datepicker.data-api touchstart.datepicker.data-api', 'table.calendar .year > .previous', DatePicker.prototype.previousYear)
				.on('click.datepicker.data-api touchstart.datepicker.data-api', 'table.calendar .year > .next', DatePicker.prototype.nextYear)
				.on('click.datepicker.data-api touchstart.datepicker.data-api', 'table.calendar td:not[.off]', DatePicker.prototype.select)
		})
	
	}(window.jQuery);