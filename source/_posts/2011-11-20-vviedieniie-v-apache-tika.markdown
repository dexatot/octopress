---
layout: post
title: "Введение в Apache Tika"
date: 2011-11-20 01:41
comments: true
categories: [JRruby, Apache]
---
## Что такое Tika?

The Apache Tika — это свободно распростроняемый инструмент использующий библиотеку парсеров для определения и извлечения метаданных и структурированного текста из различных форматов файлов. 

Tika так же ранее был под проектом Apache Lucene.

Основная идея в парсерах очень проста, все классы должны имплементиться от единого интерфейса **Parser** и каждый должен содержать метод **parse**.

Перейдем к примерам и немного поиграемся с Tika.

``` ruby example_tika.rb
# coding: utf-8

require 'java'
require "uri"

## подключаем все библиотеки из папки libs
Dir[ File.join( File.dirname(__FILE__), "../libs/*.jar" ) ].each { |jar| require jar }

##[start] java imports
import java.io.FileInputStream

import org.apache.tika.Tika
import org.apache.tika.metadata.Metadata
import org.apache.tika.sax.BodyContentHandler
import org.apache.tika.sax.XHTMLContentHandler 
import org.apache.tika.parser.html.HtmlParser
import org.apache.tika.parser.ParseContext
import org.apache.tika.parser.jpeg.JpegParser
import org.apache.tika.parser.AutoDetectParser

import org.xml.sax.helpers.DefaultHandler
##[end] java imports


class TikaExamples
	
	## конструктор для инициализации переменных класса
	def initialize
		@tika = Tika.new
	end

	## метод для извлечения метаданных из html документа
	def parse_metadata
		input = FileInputStream.new("../docs/rsreu.html")
		handler = BodyContentHandler.new
		metadata = Metadata.new
		HtmlParser.new.parse(input, handler, metadata, ParseContext.new)
		input.close()

		## метод names вощвращает массив досступных полей
		metadata.names.each { |i| puts "#{i} = #{metadata.get(i)}" }
	end
	
	## возвращаем только текст из распарсенного url
	def parse_text	
		## возвращаем только текст из распарсенного url	
		text = @tika.parseToString(java.net.URL.new("http://www.rsreu.ru"))
		puts text
	end

	## парсер структурированного текста
	def parse_html

		input = FileInputStream.new("../docs/rsreu.html")
		# getClass().getResourceAsStream("../docs/rsreu.html")
		handler = BodyContentHandler.new		
		metadata = Metadata.new
		context = ParseContext.new
		HtmlParser.new.parse(input, handler, metadata, context)
		input.close()
	end

	## парсер метаданных из изображения
	def parse_image_metadata
		input = FileInputStream.new("../docs/IMG_7993.JPG")
		metadata = Metadata.new
        parser = AutoDetectParser.new
 
    	mimeType = @tika.detect(input)
        metadata.set(Metadata::CONTENT_TYPE, mimeType)
        parser.parse(input,DefaultHandler.new,metadata,ParseContext.new)

        metadata.names.each { |i| puts "#{i} = #{metadata.get(i)}" }

	end

	def parse_pdf_metadata
		stream = FileInputStream.new("../docs/cemail.pdf")

		metadata = Metadata.new;     
        parser = AutoDetectParser.new
        handler = BodyContentHandler.new
        context = ParseContext.new     

		parser.parse(stream,handler,metadata,context);

        metadata.names.each { |i| puts "#{i} = #{metadata.get(i)}" }        
	end

	def parse_pdf_plaintext
		stream = FileInputStream.new("../docs/cemail.pdf")

		metadata = Metadata.new     
        parser = AutoDetectParser.new
        handler = BodyContentHandler.new
        context = ParseContext.new     

		parser.parse(stream,handler,metadata,context);

		plaintext = handler.toString()
	end
end
``` 