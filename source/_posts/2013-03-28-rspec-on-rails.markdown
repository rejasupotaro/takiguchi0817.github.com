---
layout: post
title: "はじめてのRspec on Rails(コントローラ編)"
date: 2013-03-28 22:23
comments: true
categories: 
---

はじめてのRspec on Rails

## セットアップ

{% codeblock %}
$ gem install rspec-rails

$ rails g rspec:controller "テスト対象のコントローラ名"
      create  spec/controllers/foo_controller_spec.rb

$ rake spec
{% endcodeblock %}

Railsのコントローラやモデルをテストするためにrspec-railsを入れて、rake specするまで。
この時点では何も起こりません。

続いてDBの設定をします。

{% codeblock %}
$ vim config/database.yml

test:
  adapter: mysql2
  encoding: utf8
  reconnect: false
  database: app_database_test
  pool: 5
  username: username
  password: password
  host: localhost

$ rake db:create RAILS_ENV=test
{% endcodeblock %}

## コントローラのテスト

あまり良い例じゃありませんが

{% codeblock lang:ruby spec/controllers/images_controller_spec.rb %}
require 'spec_helper'

describe ImagesController do
  fixtures :images

  describe 'GET #index' do
    context 'when access to route' do
      before do
        get :index
      end
      it 'responds successfully with an HTTP 200 status code' do
        expect(response).to be_success
      end
      it 'renders the index template' do
        expect(response).to render_template('index')
      end
      it 'loads all of the images into @image_list' do
        image1, image2 = Image.find(:all)
        expect(assigns(:image_list)).to match_array([image1, image2])
      end
    end
  end

  describe 'GET #show' do
    context 'when access to exsting image id' do
      before do
        get :show, :id => 1
      end
      it "responds successfully with an HTTP 200 status code" do
        expect(response).to be_success
      end
      it "renders the show templage" do
        expect(response).to render_template("show")
      end
    end

    context "when access to not exsting image id" do
      before do
        get :show, :id => 9999
      end
      it "renders the notfound template" do
        expect(response) render_template("notfound")
      end
    end
  end

end
{% endcodeblock %}

とりあえず expect(object).to あるいは not_to に

- be_success
- eq
- match_array
- be_a_new
- render_template
- redirect_to
- route_to
- be_routable
- have_selector
- include
- match

などなど、期待する値を書いていく感じですね。
(ちなみにshouldはdeprecatedになったとのことです)

## link
- [rspec-rails](https://github.com/rspec/rspec-rails)
- [rails-expectations](https://github.com/rspec/rspec-expectations)
