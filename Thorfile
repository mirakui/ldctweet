# vim:filetype=ruby
require "bundler/setup"
require "basepath"

class Ldc < Thor
  desc "post", ""
  def post
    title = CGI.unescapeHTML(recent_entry["description"])
    url = recent_entry["href"]
    if url == recent_posted_url
      exit 1
    else
      self.recent_posted_url = url
    end
    msg = "#{title} #{url}"
    if msg.length > 140
      msg = "#{title} #{bitly.shorten(url).short_url}"
    end
    hatenabm.post(:title => title, :link => url)
    twitter.update msg
    say msg
  end

  private
  def pit
    @pit ||= Pit.get("ldc", :require => {
      "twitter_consumer_key"    => "",
      "twitter_consumer_secret" => "",
      "twitter_access_token"    => "",
      "twitter_access_secret"   => "",
      "ldc_account"             => "",
      "ldc_api_key"             => "",
      "bitly_account"           => "",
      "bitly_api_key"           => "",
      "hatenabm_account"        => "",
      "hatenabm_password"       => ""
    })
  end
  
  def twitter
    @twitter ||= TwitterOAuth::Client.new(
      :consumer_key    => pit["twitter_consumer_key"],
      :consumer_secret => pit["twitter_consumer_secret"],
      :token           => pit["twitter_access_token"],
      :secret          => pit["twitter_access_secret"]
    )
  end

  def ldc
    @ldc ||= Ldclip.new(pit["ldc_account"], pit["ldc_api_key"])
  end

  def recent_entry
    @recent_entry ||= Nokogiri.parse(ldc.get) % "post[1]"
  end

  def recent_posted_url
    RECENT_POSTED_URL_PATH.exist? ? RECENT_POSTED_URL_PATH.open.read.chomp : nil
  end

  def recent_posted_url=(url)
    RECENT_POSTED_URL_PATH.open("w").puts url
  end

  def bitly
    @bitly ||= Bitly::V3.new(pit["bitly_account"], pit["bitly_api_key"])
  end

  def hatenabm
    @hatenabm ||= HatenaBM.new(
      :user => pit["hatenabm_account"],
      :pass => pit["hatenabm_password"])
  end
end

class Config < Thor
  desc "init_twitter", ""
  def init_twitter
    consumer_key = ask 'Consumer Key>'
    consumer_secret = ask 'Consumer Secret>'

    t = TwitterOAuth::Client.new(
      :consumer_key => consumer_key,
      :consumer_secret => consumer_secret
    )

    req = t.request_token

    say 'OK', :green
    say "please access and get PIN: #{req.authorize_url}", :yellow
    pin = ask('PIN>').to_i
    say pin

    acc = t.authorize(
      req.token,
      req.secret,
      :oauth_verifier => pin
    )

    say "Authorized:    #{t.authorized?}", :green
    say "Access Token:  #{acc.token}", :green
    say "Access Secret: #{acc.secret}", :green
  end
end
