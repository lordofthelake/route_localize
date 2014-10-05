# Route Localize

Rails 4 engine to translate routes using locale files and subdomains.


## Install

In your Rails application's `Gemfile` add:

```rb
gem "route_localize", github: "sunny/route_localize"
```

Install the plugin by running:

```sh
$ bundle
```

## Scopes

Route Localize adds two routing scopes you can use on your routes:

- `localize`: your locale is the first thing in the path (`http://example.com/en/foo`)
- `localize_subdomain`: your locale is your subdomain (`http://en.example.com/foo`)


## Usage

In your `config/routes.rb`, localize some routes with one of the scopes. For example:

```rb
scope localize: [:en, :fr] do
  get 'trees/new', to: 'trees#new'
end
root 'pages#index'
```

Create a `config/locales/routes.yml` with translations for each part of your routes under the `routes` key. For example:

```yml
fr:
  routes:
    trees: arbres
    new: nouveau
```

With this example you would have the following routes defined:

            Prefix Verb URI Pattern               Controller#Action
      trees_new_en GET /trees/new(.:format)      trees#new {:subdomain=>:en}
      trees_new_fr GET /arbres/nouveau(.:format) trees#new {:subdomain=>:fr}

You also get the `trees_new_path` and `trees_new_url` helpers that will call
`trees_new_en` or `trees_new_fr` depending on the current locale.


## Language switcher

If you want to be able to switch to the current page in another language
add the following inside your `app/helpers/application_helper.rb`:

```rb
module ApplicationHelper
  include RouteLocalizeHelper
end
```

You can then use the `locale_switch_url` or `locale_switch_subdomain_url` helpers in your views:

```erb
<%= link_to "fr", locale_switch_url("fr") %>
<%= link_to "en", locale_switch_url("en") %>
```


### Change the parameters in your switcher

If your params are different depending on the language, you can override
the switcher's params by creating a `route_localize_options` method that
takes the locale as a parameter.

For example if you would like to switch from
`http://en.example.org/products/keyboard`
to `http://fr.example.org/produits/clavier`, where `keyboard` and `clavier`
are the `:id` parameter.

In this case you might already have this in controller:

```rb
class ProductsController < ApplicationController
  def show
    if I18n.locale = "fr"
      @tree = Product.find_by_name_fr(params[:id])
    else
      @tree = Product.find_by_name_en(params[:id])
    end
  end
end
```

Then you would need to add this inside your controller:

```
  helper_method :route_localize_path_options
  def route_localize_path_options(locale)
    { id: (locale == "fr" ? @tree.name_fr : @tree.name_en) }
  end
```


## Caveats

Rails' `url_for` cannot find the translation url automatically,
prefer to use the `_path` and `_url` methods instead.

If you can't, one way around is to use `RouteLocalize.translate_path`.

For example :

```ruby
RouteLocalize.translate_path(url_for(controller: 'trees', action: 'index'),
                             I18n.locale)
```

If you are using subdomains you should add `by_subdomain: true` option.


## Development

You may help by [submitting issues](https://github.com/sunny/route_localize),
creating pull requests, talking about the gem or by saying thanks.


## Other gems to translate Rails routes

The following gems could also be a good match for your project:

- [translate_routes](https://github.com/raul/translate_routes)
- [route_translator](https://github.com/enriclluelles/route_translator/)
- [rails-translate-routes](https://github.com/francesc/rails-translate-routes/)
- [routing-filter](https://github.com/svenfuchs/routing-filter)

Route Localize is different from these solutions because it:

- can add a constraint to the subdomain instead of relying on the locale beeing in the url (`en/…` `fr/`)
- plays well with gems that introduce extra locales, routes you don't want to translate, or reload routes before i18n is loaded (`activeadmin` for example)
- includes a language switcher helper that returns the correct url in each other language

