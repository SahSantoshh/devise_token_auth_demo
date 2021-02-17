# README

Devise token auth is not working as aspected

- Gem Details

  - `ruby '2.6.5'`
  - `gem 'rails', '~> 6.0.3', '>= 6.0.3.5'`
  - `gem 'rails', '~> 6.0.3', '>= 6.0.3.5'`
  - `gem 'devise_token_auth', '~> 1.1', '>= 1.1.4'`

- User Model

```ruby
  class User < ActiveRecord::Base
    extend Devise::Models
    # Include default devise modules. Others available are:
    # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
    devise :database_authenticatable, :registerable,
          :recoverable, :rememberable, :validatable
    include DeviseTokenAuth::Concerns::User
  end
```

- Application Controller

```ruby
  class ApplicationController < ActionController::API
    include DeviseTokenAuth::Concerns::SetUserByToken
  end
```

- Application.rb

```ruby
...
...
Bundler.require(*Rails.groups)

module DeviseTokenAuthDemo2
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 6.0

    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration can go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded after loading
    # the framework and any gems in your application.

    # Only loads a smaller set of middleware suitable for API only apps.
    # Middleware like session, flash, cookies can be added back manually.
    # Skip views, helpers and assets when generating a new resource.
    config.api_only = true
    config.middleware.use Rack::Cors do
      allow do
        origins '*'
        resource '*',
                 headers: :any,
                 expose: %w[access-token expiry token-type uid client],
                 methods: %i[get post options delete put]
      end
    end
  end
end
```

## One Scaffold to Test

```ruby
class TweetsController < ApplicationController
  before_action :authenticate_user!
  before_action :set_tweet, only: %i[show update destroy]

  # GET /tweets
  def index
    @tweets = Tweet.all

    render json: @tweets
  end

  # GET /tweets/1
  def show
    render json: @tweet
  end

  # POST /tweets
  def create
    @tweet = Tweet.new(tweet_params)

    if @tweet.save
      render json: @tweet, status: :created, location: @tweet
    else
      render json: @tweet.errors, status: :unprocessable_entity
    end
  end

  # PATCH/PUT /tweets/1
  def update
    if @tweet.update(tweet_params)
      render json: @tweet
    else
      render json: @tweet.errors, status: :unprocessable_entity
    end
  end

  # DELETE /tweets/1
  def destroy
    @tweet.destroy
  end

  private

  # Use callbacks to share common setup or constraints between actions.
  def set_tweet
    @tweet = Tweet.find(params[:id])
  end

  # Only allow a trusted parameter "white list" through.
  def tweet_params
    params.require(:tweet).permit(:title, :desc)
  end
end
```

## Postman Details

Request

```curl
curl --location --request GET 'http://localhost:3000/tweets' \
--header 'client: otDRP7XKpxwlw3-6t_FUlQ' \
--header 'uid: sahsantoshh@gmail.com' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer r3sGytCe-q7No4ZCnaDdSA'
```

Response

`status: 401: Unauthorized`

```json
{
  "errors": ["You need to sign in or sign up before continuing."]
}
```

StackTrace

```shell
Started GET "/tweets" for ::1 at 2021-02-17 14:39:34 +0545
Processing by TweetsController#index as */*
  Parameters: {"tweet"=>{}}
Filter chain halted as :authenticate_user! rendered or redirected
Completed 401 Unauthorized in 17ms (Views: 0.2ms | ActiveRecord: 1.0ms | Allocations: 6753)
```
