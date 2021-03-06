---
layout: post
title: "Symfony Tutorial: Building a Blog (Part 2)"
description: "Let's create a secure blog engine with Symfony."
longdescription: "Creating applications with Symfony is easy and can be scaled to be used in any requirement. The tools that it provides to create and maintain web applications is amazing and replaces repetitive tasks. Let's use Symfony to create a blog engine."
date: 2018-01-04 08:30
category: Technical Guide, PHP, Symfony
author:
  name: Greg Holmes
  url: https://github.com/GregHolmes
  mail: iam@gregholmes.co.uk
  avatar: "https://avatars0.githubusercontent.com/u/2411269?s=460&v=4"
design:
  bg_color: "#000000"
  image: https://cdn.auth0.com/blog/symfony-blog/logo.png
tags:
- symfony
- php
- auth0
- bootstrap
- authentication
- web-app
related:
- 2017-12-28-symfony-tutorial-building-a-blog-part-1
- 2016-06-23-creating-your-first-laravel-app-and-adding-authentication
---

**TL;DR:** Symfony is a PHP framework as well as a set of reusable PHP components and libraries. It uses the Model-View-Controller design pattern and can be scaled to be used in any requirement. It aims to speed up the creation and maintenance of web applications, replacing repetitive code. In this part of the article, we will cover installing [Bootstrap, a UI framework for web applications](https://getbootstrap.com/), to make the blog engine look nicer visually. The final code can be found at this [repository](https://github.com/auth0-blog/symfony-blog-part-2).

---

## Symfony Tutorial: About Part 1 and Part 2

[In the first article](https://auth0.com/blog/symfony-tutorial-building-a-blog-part-1/), we:

* installed and configured a Symfony installation;
* created two new database tables: `author` and `blog_post`;
* allowed users to authenticate with [Auth0](https://auth0.com);
* and ensured that the authenticated users have `Author` instances associated before using the system.

In this part of the article, we will cover installing [Bootstrap, a UI framework for web applications](https://getbootstrap.com/), to make the blog engine look nicer visually. We will also enhance our blog engine to allow visitors to:

* see a list of blog posts;
* read a specific blog post;
* and find out more about authors.

Besides that, authenticated authors will be able to:

* create a new blog post;
* see all of their own blog posts;
* and delete their own blog posts from the system.

## Building the Blog Engine

### Before Starting

Make sure you have followed all instructions in the first part. However, if for some reason you lost the code created in the first part, or if you want to start here, [feel free to clone this GitHub repository](https://github.com/auth0-blog/symfony-blog-part-1). The following commands will set up the application for you:

```bash
git clone https://github.com/auth0-blog/symfony-blog-part-1
cd symfony-blog-part-1
```

After that, you will have to create a file called `.env` in the project root and paste the following into it:

```yml
DATABASE_HOST={DATABASE_HOST}
DATABASE_PORT={DATABASE_PORT}
DATABASE_NAME={DATABASE_NAME}
DATABASE_USER={DATABASE_USER}
DATABASE_PASSWORD={DATABASE_PASSWORD}
AUTH0_CLIENT_ID={AUTH0_CLIENT_ID}
AUTH0_CLIENT_SECRET={AUTH0_CLIENT_SECRET}
AUTH0_DOMAIN={AUTH0_DOMAIN}
```

Note that you will have to replace the values above. [Check the first part to understand how to replace them](https://auth0.com/blog/symfony-tutorial-building-a-blog-part-1/).

__Pro Tip!__ If you do not have a MySQL database available, an easy way to bootstrap one is with Docker:

```bash
docker run --name symfony-blog-mysql \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=myextremellysecretpassword \
    -e MYSQL_DATABASE=symfony-blog \
    -e MYSQL_USER=symfony-blog-user \
    -e MYSQL_PASSWORD=mysecretpassword \
    -d mysql:5.7
```

The last thing you will need to do is to use composer to install the dependencies:

```bash
composer install
```

This will trigger a series of questions that you can answer as follows:

```bash
database_host (127.0.0.1): 127.0.0.1
database_port (null): 3306
database_name (symfony): symfony-blog
database_user (root): symfony-blog-user
database_password (null): mysecretpassword
```

For the questions related to `mailer_transport` and `secret`, you can simply press `Enter` to accept the default values.

Last thing, if you haven't followed the first part of this series, you might need to issue the following commands to create the database tables and to populate them:

```bash
php bin/console doctrine:migrations:migrate
php bin/console doctrine:fixtures:load
```

### Installing Bootstrap

In order to install [Bootstrap](https://getbootstrap.com/), we need [Symfony's Webpack Encore](https://github.com/symfony/webpack-encore), which is a simpler way to integrate [Webpack](https://webpack.js.org/) into your application.

__NOTE__ If you do not have [Yarn](https://yarnpkg.com), a Javascript package manager, installed, you will need to install and configure this first. So go to their [Installation](https://yarnpkg.com/lang/en/docs/install/) page and follow the instructions for installing and configuring Yarn first.

You can install [Symfony's Webpack Encore](https://github.com/symfony/webpack-encore) by running the following command:

```bash
yarn add @symfony/webpack-encore --dev
```

In the root directory of the project there will be 2 new files (`package.json`, `yarn.lock`) and a new directory (`node_modules`).

Create `webpack.config.js` in the root of the project, this is just the file that contains all of the web pack configurations.

Paste the following code into this file:

```JavaScript
// webpack.config.js
var Encore = require('@symfony/webpack-encore');

Encore
    .setOutputPath('web/build/')
    .setPublicPath('/build')
    .cleanupOutputBeforeBuild()
    .addEntry('app', './assets/js/main.js')
    .addStyleEntry('global', './assets/css/global.scss')
    .enableSassLoader()
    .autoProvidejQuery()
    .enableSourceMaps(!Encore.isProduction());

module.exports = Encore.getWebpackConfig();
```

The configuration above uses two assets: `main.js` and `global.scss`. Let's create a new directory called `assets` in the project root, and there we will need to create two subdirectories: `js` and `css`. Inside these subdirectories, let's create the `main.js` and `global.scss` files. We will populate them further in the tutorial. After that, we will have the following substructure:

* `./assets/js/main.js`
* `./assets/css/global.scss`

Before we can compile our JavaScript and SCSS files to be used in our Symfony templates, we will need to install some dependencies. [`sass-loader`](https://github.com/webpack-contrib/sass-loader) and [`node-sass`](https://github.com/sass/node-sass) are libraries that load SASS/SCSS files and compile them to CSS. Let's install these dependencies by issuing the following codes:

```bash
yarn add sass-loader node-sass --dev
```

Now, we can compile our Javascript and CSS into assets to be used by Symfony by running this command, `yarn run encore dev --watch`.

Open the base twig template (which can be found at `./app/Resources/views/base.html.twig`) and replace the contents with:

{% highlight html %}
{% raw %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>{% block title %}Welcome!{% endblock %}</title>
    {% block stylesheets %}
        <link rel="stylesheet" href="{{ asset('build/global.css') }}" />
    {% endblock %}
</head>
<body>
{% block body %}{% endblock %}
{% block javascripts %}
    <script src="{{ asset('build/app.js') }}"></script>
{% endblock %}
</body>
</html>
{% endraw %}
{% endhighlight %}

We've now included both compiled CSS and empty Javascript files into the base template to be used throughout our app!

In order to make use of Bootstrap we need to install jQuery, so run the command:

```bash
yarn add jquery --dev
```

Once installed, at the top of the empty `./assets/js/main.js` file, insert `var $ = require('jquery');`.

We want an app CSS asset file. Let's create the following file `assets/css/main.scss`. Then, in `assets/js/main.js` at the bottom paste the following: `require('../css/main.scss');`.

Let's move the contents of our CSS file we created in part 1 (`web/css/style.css`) into the new file we've created above (`assets/css/global.scss`).

Finally, in our `base.html.twig`, in the Stylesheets block, paste the following: `{% raw %}<link rel="stylesheet" href="{{ asset('build/app.css') }}">{% endraw %}`.

Now we need to install [`bootstrap-sass`](https://github.com/twbs/bootstrap-sass) with the following command: `yarn add bootstrap-sass --dev`. We need to import this into our Sass file. So in `./assets/css/global.scss`, let's insert the following lines:

```css
$brand-primary: darken(#428bca, 20%);

@import '~bootstrap-sass/assets/stylesheets/bootstrap';
```

We're going to want to override the path to icons by loading Bootstrap in the `global.scss` file:

```css
$icon-font-path: "~bootstrap-sass/assets/fonts/bootstrap/";
```

Finally in your `main.js` file you need to require bootstrap-sass under your `var $ = require('jquery');` line:

```JavaScript
require('bootstrap-sass');
```

You've now set up Bootstrap to be used in your Symfony Blog.

{% include tweet_quote.html quote_text="Integrating Webpack, Symfony, and Bootstrap is really easy!" %}

### Showing Blog Posts

Let's create our blog controller by running the following command:

```bash
php bin/console generate:controller
```

![Creating Blog Controller with Symfony](https://cdn.auth0.com/blog/symfony-part-2/creating-blog-controller.png)

If you follow the instructions as shown in the image above, you'll find that you have a new Controller class in `./src/AppBundle/Controllers/` called `BlogController`. You'll also have three new templates in `./src/AppBundle/Resources/views/Blog/`.

__NOTE__: If you cannot see the image, the full controller can be found [here](https://github.com/GregHolmes/symfony-blog/blob/master/part-2/src/AppBundle/Controller/BlogController.php)

Delete the DefaultController (`./src/AppBundle/Controllers/DefaultController.php`) as it's not needed.

Back in our `AdminController`, we need to add a route on the controller itself. Because there may be conflicts of routes between the two controllers. So above `class AdminController extends Controller` add:

```php
/**
 * @Route("/admin")
 */
 ```

Open `./src/AppBundle/Controllers/BlogController.php` and find the `entriesAction`. Then, configure the routing for this controller to be the homepage and give the action a service name. So above `public function entriesAction()` replace the annotation with:

```php
/**
 * @Route("/", name="homepage")
 * @Route("/entries", name="entries")
 */
```

This allows us to call the action by the service name, but also places the root route as displaying the entries.

We need to make use the entity manager and the repositories for the entities in order to retrieve database data. At the top of the `BlogController` class we want to inject these services.

```php
/** @var EntityManagerInterface */
private $entityManager;

/** @var \Doctrine\Common\Persistence\ObjectRepository */
private $authorRepository;

/** @var \Doctrine\Common\Persistence\ObjectRepository */
private $blogPostRepository;

/**
 * @param EntityManagerInterface $entityManager
 */
public function __construct(EntityManagerInterface $entityManager)
{
    $this->entityManager = $entityManager;
    $this->blogPostRepository = $entityManager->getRepository('AppBundle:BlogPost');
    $this->authorRepository = $entityManager->getRepository('AppBundle:Author');
}
```

As you can see there is a class declared here so we need to add it to the namespaces at the top of the file. Where it says:

```php
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
```

Add: `use Doctrine\ORM\EntityManagerInterface;` above the two.

Great, in our entire controller we can call the `blogPostRepository`, `authorRepository` or `entityManager` when needed. The first two are used for retrieving data from the database, whereas the third will be used for inserting, updating, or deleting data (it can also be used for retrieving, but by setting up the construct this way, we will be reducing duplicate code).

Let us populate the `entriesAction` function now:

```php
return $this->render('AppBundle:Blog:entries.html.twig', [
    'blogPosts' => $this->blogPostRepository->findAll()
]);
```

### Creating Blog Posts

Now that we have made a member restricted area of the site, let's allow the authenticated users to create a new blog post.

Let's create a new file called `./src/AppBundle/Form/EntryFormType.php` and paste the following into there:

```php
<?php

namespace AppBundle\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Validator\Constraints\NotBlank;

class EntryFormType extends AbstractType
{
    /**
     * {@inheritdoc}
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add(
                'title',
                TextType::class,
                [
                    'constraints' => [new NotBlank()],
                    'attr' => ['class' => 'form-control']
                ]
            )
            ->add(
                'slug',
                TextType::class,
                [
                    'constraints' => [new NotBlank()],
                    'attr' => ['class' => 'form-control']
                ]
            )
            ->add(
                'description',
                TextareaType::class,
                [
                    'constraints' => [new NotBlank()],
                    'attr' => ['class' => 'form-control']
                ]
            )
            ->add(
                'body',
                TextareaType::class,
                [
                    'constraints' => [new NotBlank()],
                    'attr' => ['class' => 'form-control']
                ]
            )
            ->add(
                'create',
                SubmitType::class,
                [
                    'attr' => ['class' => 'form-control btn-primary pull-right'],
                    'label' => 'Create!'
                ]
            );
    }

    /**
     * {@inheritdoc}
     */
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'data_class' => 'AppBundle\Entity\BlogPost'
        ]);
    }

    /**
     * {@inheritdoc}
     */
    public function getName()
    {
        return 'author_form';
    }
}
```

As you can see, the data class to be used is the entity `BlogPost`, and if you compare the fields in `buildForm` you'll notice the first argument of each, the name, matches the names of the properties in BlogPost entity.

A new controller method is needed, so add this function into your `AdminController` class:

```php
/**
 * @Route("/create-entry", name="admin_create_entry")
 *
 * @param Request $request
 *
 * @return \Symfony\Component\HttpFoundation\Response
 */
public function createEntryAction(Request $request)
{
    $blogPost = new BlogPost();

    $author = $this->authorRepository->findOneByUsername($this->getUser()->getUserName());
    $blogPost->setAuthor($author);

    $form = $this->createForm(EntryFormType::class, $blogPost);
    $form->handleRequest($request);

    // Check is valid
    if ($form->isValid()) {
        $this->entityManager->persist($blogPost);
        $this->entityManager->flush($blogPost);

        $this->addFlash('success', 'Congratulations! Your post is created');

        return $this->redirectToRoute('admin_entries');
    }

    return $this->render('AppBundle:Admin:entry_form.html.twig', [
        'form' => $form->createView()
    ]);
}
```

At the top in the namespaces, we need to add the two new classes we're using: `BlogPost` and `EntryFormType` so paste:

```php
use AppBundle\Entity\BlogPost;
use AppBundle\Form\EntryFormType;
```

We now need the template, so create a new file called `./src/AppBundle/Resources/views/Admin/entry_form.html.twig` and insert the following code into it:

{% highlight html %}
{% raw %}
{% extends '::base.html.twig' %}

{% block title %}{% endblock %}

{% block body %}
    <div class="container">
        <div class="blog-header">
            <h2 class="blog-title"></h2>
        </div>

        <div class="row">
            <div class="col-sm-12 blog-main">
                {% for label, messages in app.flashes %}
                    {% for message in messages %}
                        <div class="bg-{{ label }}">
                            {{ message }}
                        </div>
                    {% endfor %}
                {% endfor %}

                {{ form_start(form) }}
                    <div class="col-md-12">
                        <div class="form-group col-md-4">
                            {{ form_row(form.title) }}
                        </div>
                        <div class="form-group col-md-4">
                            {{ form_row(form.slug) }}
                        </div>
                        <div class="form-group col-md-12">
                            {{ form_row(form.description) }}
                        </div>
                        <div class="form-group col-md-12">
                            {{ form_row(form.body) }}
                        </div>
                        <div class="form-group col-md-4 pull-right">
                            {{ form_widget(form.create) }}
                        </div>
                    </div>
                {{ form_end(form) }}
            </div>
        </div>
    </div>
{% endblock %}
{% endraw %}
{% endhighlight %}

Before we try to create a new entry, let's build the page that displays all of the authenticated users blog posts.

### Displaying Blog Posts Created by Authenticated Author

In your `AdminController`, let's add a new method called `entriesAction()` and input the code below. All this will do is retrieve all of the blog posts by the authenticated user and pass those into the template `entries.html.twig` to be displayed.

```php
/**
 * @Route("/", name="admin_index")
 * @Route("/entries", name="admin_entries")
 *
 * @return \Symfony\Component\HttpFoundation\Response
 */
public function entriesAction()
{
    $author = $this->authorRepository->findOneByUsername($this->getUser()->getUserName());

    $blogPosts = [];

    if ($author) {
        $blogPosts = $this->blogPostRepository->findByAuthor($author);
    }

    return $this->render('AppBundle:Admin:entries.html.twig', [
        'blogPosts' => $blogPosts
    ]);
}
```

Time to create the template `./src/AppBundle/Resources/views/Admin/entries.html.twig` to store the following code in:

{% highlight html %}
{% raw %}
{% extends '::base.html.twig' %}

{% block title %}{% endblock %}

{% block body %}
    <div class="container">
        <div class="blog-header">
            <h1 class="blog-title">Author admin</h1>
            <p class="lead blog-description"></p>
        </div>

        <div class="row">
            <div class="col-md-12 col-lg-12 col-xl-12">
                {% for label, messages in app.flashes %}
                    {% for message in messages %}
                        <div class="alert alert-{{ label }}" role="alert">
                            <span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
                            {{ message }}
                        </div>
                    {% endfor %}
                {% endfor %}
            </div>
            <div class="col-md-12 col-lg-12 col-xl-12">
                <a type="button" href="{{ path('admin_create_entry') }}" class="btn btn-primary pull-right">Add Entry</a>
            </div>
            <div class="col-sm-12 blog-main">
                <table class="table table-striped">
                    <thead>
                        <th>Title</th>
                        <th>Created At</th>
                        <th>Updated At</th>
                    </thead>
                    {% for blogPost in blogPosts %}
                        <tr>
                            <td>{{ blogPost.title }}</td>
                            <td>{{ blogPost.createdAt|date('F j, Y') }}</td>
                            <td>{{ blogPost.updatedAt|date('F j, Y') }}</td>
                        </tr>
                    {% else %}
                        <tr>
                            <td colspan="5">No entries available</td>
                        </tr>
                    {% endfor %}
                </table>
            </div>
        </div>
    </div>
{% endblock %}
{% endraw %}
{% endhighlight %}

As you can see in this template, there is a button to "Add entry" which will direct the user to create a new entry when clicked.

### Creating Delete Functionality for Author's Posts

Our next step is to delete the authenticated users blog posts on demand. So in `AdminController` create a new method with the following code:

```php
/**
 * @Route("/delete-entry/{entryId}", name="admin_delete_entry")
 *
 * @param $entryId
 *
 * @return \Symfony\Component\HttpFoundation\RedirectResponse
 */
public function deleteEntryAction($entryId)
{
    $blogPost = $this->blogPostRepository->findOneById($entryId);
    $author = $this->authorRepository->findOneByUsername($this->getUser()->getUserName());

    if (!$blogPost || $author !== $blogPost->getAuthor()) {
        $this->addFlash('error', 'Unable to remove entry!');

        return $this->redirectToRoute('admin_entries');
    }

    $this->entityManager->remove($blogPost);
    $this->entityManager->flush();

    $this->addFlash('success', 'Entry was deleted!');

    return $this->redirectToRoute('admin_entries');
}
```

This will check if the entryId passed in exists, check to ensure the authenticated user is the author of the article and then delete it.

There is no template needed for this, however, we still need somewhere in the templates to show the action. So in `./src/AppBundle/Resources/views/Admin/entries.html.twig` let's make some additions.

In the table headers, let's add a new row. This will this element change from:

```html
<thead>
    <th>Title</th>
    <th>Created At</th>
    <th>Updated At</th>
</thead>
```

to:

```html
<thead>
    <th>Title</th>
    <th>Created At</th>
    <th>Updated At</th>
    <th>Action</th>
</thead>
```

And in the for loop, add a new td, which will contain the delete button. We will change it from:

{% highlight html %}
{% raw %}
<td>{{ blogPost.title }}</td>
<td>{{ blogPost.createdAt|date('F j, Y') }}</td>
<td>{{ blogPost.updatedAt|date('F j, Y') }}</td>
{% endraw %}
{% endhighlight %}

to:

{% highlight html %}
{% raw %}
<td>{{ blogPost.title }}</td>
<td>{{ blogPost.createdAt|date('F j, Y') }}</td>
<td>{{ blogPost.updatedAt|date('F j, Y') }}</td>
<td><a class="btn btn-danger" href="{{ path('admin_delete_entry', {'entryId': blogPost.id}) }}">Delete</a></td>
{% endraw %}
{% endhighlight %}

### Add Pagination to Blog Posts List

We don't want to be loading all blog posts into the page, so let's add some pagination.

In the `./src/AppBundle/Controllers/BlogController.php` find `entriesAction()` and within the empty brackets type in: `Request $request`

At the top of the controller we need to include this class so where it shows:

```php
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Doctrine\ORM\EntityManagerInterface;
```

Paste the following below those:

```php
use Symfony\Component\HttpFoundation\Request;
```

`Request` allows you to gather data, for example, if there are any parameters in a POST or GET request.

At the top of our BlogController class, let's add a blog post limit constant:

```php
/** @var integer */
const POST_LIMIT = 5;
```

We will want to default the page number to 1, but then make use of the object `$request` to find out if the user is on a different page number. So in the top of your `entriesAction` place the following:

```php
$page = 1;

if ($request->get('page')) {
    $page = $request->get('page');
}
```

This data is useless to us unless `BlogPostRepository` knows what to do with it. When we created the `BlogPost` entity, it also created a repository for us. This repository contains our methods for custom queries or more in depth queries. So open: `./src/AppBundle/Repository/BlogPostRepository.php`

The first method we're going to need is to get all of the posts based on the page number and the limit previously set in the `BlogController`. The method below does exactly that. It retrieves the paginated number of blog posts.

```php
/**
 * @param int $page
 * @param int $limit
 *
 * @return array
 */
public function getAllPosts($page = 1, $limit)
{
    $entityManager = $this->getEntityManager();
    $queryBuilder = $entityManager->createQueryBuilder();
    $queryBuilder
        ->select('bp')
        ->from('AppBundle:BlogPost', 'bp')
        ->setFirstResult($limit * ($page - 1))
        ->setMaxResults($limit);

    return $queryBuilder->getQuery()->getResult();
}
```

We're going to need another method in order to get the count of blog posts so that our pagination can know whether you're on the last page or not. So below the previously added method, add another:

```php
/**
 * @return array
 */
public function getPostCount()
{
    $entityManager = $this->getEntityManager();
    $queryBuilder = $entityManager->createQueryBuilder();
    $queryBuilder
        ->select('count(bp)')
        ->from('AppBundle:BlogPost', 'bp');

    return $queryBuilder->getQuery()->getSingleScalarResult();
}
```

Back in the `entriesAction` in your `BlogController` we're going to need to make use of those methods.

Your return previously looked like this:

```php
return $this->render('AppBundle:Blog:entries.html.twig', [
    'blogPosts' => $this->blogPostRepository->findAll()
]);
```

Let's change it to call those methods and pass the data as well as the page number and post limit into the template:

```php
return $this->render('AppBundle:Blog:entries.html.twig', [
    'blogPosts' => $this->blogPostRepository->getAllPosts($page, self::POST_LIMIT),
    'totalBlogPosts' => $this->blogPostRepository->getPostCount(),
    'page' => $page,
    'entryLimit' => self::POST_LIMIT
]);
```
Time to show your blog posts in a template file. Open `./src/AppBundle/Resources/views/Blog/entries.html.twig` then change the title to whatever you wish (I changed it to "Blog Posts"). Inside `{% raw %}{% block body %}{% endraw %}`, paste the following code:

{% highlight html %}
{% raw %}
    <div class="container">
        <div class="blog-header">
            <h1 class="blog-title">Blog tutorial</h1>
            <p class="lead blog-description">A basic description of the blog, built in Symfony, styled in Bootstrap 3, secured by <a href="http://auth0.com">Auth0</a>.</p>
        </div>

        <div class="row">
            <div class="col-sm-8 blog-main">
                {% for blogPost in blogPosts %}
                    {% set paragraphs = blogPost.description|split('</p>') %}
                    {% set firstParagraph = paragraphs|first ~ '</p>' %}
                    <div class="blog-post">
                        <h2 class="blog-post-title">
                            {{ blogPost.title }}
                        </h2>
                        <p class="blog-post-meta">
                            {{ blogPost.getUpdatedAt|date('F j, Y') }} by

                            {% if blogPost.author %}
                                {{ blogPost.author.name }}
                            {% else %}
                                Unknown Author
                            {% endif %}
                        </p>
                        {{ firstParagraph|raw }}<br />
                    </div>
                {% else %}
                    <div class="alert alert-danger" role="alert">
                        <span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
                        <span class="sr-only">Error:</span>
                        You have no blog articles. Please log in and create an article.
                    </div>
                {% endfor %}
            </div>
        </div>
    </div>
{% endraw %}
{% endhighlight %}

In your `entries.html.twig` template, find `{% raw %}{% endfor %}{% endraw %}`. Below this we want to add the pagination buttons. So let's put the following code there:

{% highlight html %}
{% raw %}
{% set canPrevious = page > 1 %}
{% set canNext = (page * entryLimit) < totalBlogPosts %}
<nav>
    <ul class="pager">
        <li class="previous {% if canPrevious == false %}disabled{% endif %}">
            <a href="{% if canPrevious %}{{ path('entries', {'page': page - 1}) }}{% endif %}">
                <span aria-hidden="true">&larr;</span> Older
            </a>
        </li>
        <li class="next {% if canNext == false %}disabled{% endif %}">
            <a href="{% if canNext %}{{ path('entries', {'page': page + 1}) }}{% endif %}">
                Newer <span aria-hidden="true">&rarr;</span>
            </a>
        </li>
    </ul>
</nav>
{% endraw %}
{% endhighlight %}

The above code determines whether the user can move to a previous or a next page.

Now reload your browser, you'll see the previously shown blog post, but below that, you'll see disabled "Previous" and "Next" buttons, they're disabled because you're on page 1, and there is only 1 blog post.

> Quick tip, to start and stop your application you can use `php bin/console server:start` and `php bin/console server:stop`.

### Adding Navigation

We need to enable users to find their way through the blog. So time to add some navigation. Create a new file in `app/Resources/views/nav_bar.html.twig` and paste the following in:

{% highlight html %}
{% raw %}
<nav class="navbar navbar-default navbar-fixed-top">
    <div id="navbar" class="collapse navbar-collapse pull-right">
        <ul class="nav navbar-nav">
            {% if app.request.get('_route') not in ['homepage', 'entries'] %}
                <li><a href="{{ path("homepage") }}">Home</a></li>
            {% endif %}
            {% if app.user %}
                <li><a href="{{ path("admin_index") }}">Admin</a></li>
                <li><a href="{{ logout_url("secured_area") }}">Logout</a></li>
            {% else %}
                <li class="active"><a href="/connect/auth0">Login</a></li>
            {% endif %}
        </ul>
    </div>
</nav>
{% endraw %}
{% endhighlight %}

Then include the file into the `app/Resources/views/base.html.twig` just below the `<body>` opening tag:

{% highlight html %}
{% raw %}
{% include 'nav_bar.html.twig' %}
{% endraw %}
{% endhighlight %}

### Showing Specific Blog Post

On the `entryAction` of the `BlogController` class, let's add a service name to the action. Where it shows `* @Route("/entry/{slug}")` let's replace it by `* @Route("/entry/{slug}", name="entry")`.

Let's retrieve the blog post from the database by the given slug at the top of the method:

```php
$blogPost = $this->blogPostRepository->findOneBySlug($slug);
```

We need to3 make sure the blog post exists before passing data into and displaying the template. So let's now do a check:

```php
if (!$blogPost) {
    $this->addFlash('error', 'Unable to find entry!');

    return $this->redirectToRoute('entries');
}
```

This will set a "flash" session with an error message and redirect the user to the entries page to display the error.

Next, in the return's 2nd argument (the array), put in an entry (`'blogPost' => $blogPost`). Your final action will look like:

```php
/**
* @Route("/entry/{slug}", name="entry")
*/
public function entryAction($slug)
{
   $blogPost = $this->blogPostRepository->findOneBySlug($slug);

   if (!$blogPost) {
       $this->addFlash('error', 'Unable to find entry!');

       return $this->redirectToRoute('entries');
   }

   return $this->render('AppBundle:Blog:entry.html.twig', array(
       'blogPost' => $blogPost
   ));
}
```

Now, we need to output this data in the template. So open `./src/AppBundle/Resources/views/Blog/entry.html.twig` and, within `{% raw %}{% block body %}{% endraw %}`, we need to add some content:

{% highlight html %}
{% raw %}
<div class="container">
 <div class="blog-header">
     <h1 class="blog-title">Blog tutorial</h1>
     <p class="lead blog-description">A basic description of the blog, built in Symfony, styled in Bootstrap 3, and secured by <a href="http://auth0.com">Auth0</a>.</p>
 </div>

 <div class="row">
     <div class="col-sm-8 blog-main">
         <div class="blog-post">
             <h2 class="blog-post-title">{{ blogPost.title }}</h2>
             <p class="blog-post-meta">{{ blogPost.updatedAt|date('F j, Y') }} by {{ blogPost.author.name }}</p>
             <h3>Description:</h3>
             <p>{{ blogPost.description|raw }}</p>

             <h3>Body:</h3>
             <p>{{ blogPost.body|raw }}</p>
         </div>
     </div>
 </div>
</div>
{% endraw %}
{% endhighlight %}

The above code simply outputs the details of the blog post, its title, when it was updated at, the author, description, and the body.

Our next problem is how to access this new page we've created. Back in your `entries.html.twig` template, find `{% raw %}{{ firstParagraph|raw }}<br />{% endraw %}` and paste below:

{% highlight html %}{% raw %}<a href="{{ path('entry', {'slug': blogPost.slug}) }}">Read more</a>{% endraw %}{% endhighlight %}

Then you need to find `{% raw %}{{ blogPost.title }}{% endraw %}` and wrap this in `<a>` tags so it will look like:

{% highlight html %}
{% raw %}
<a href="{{ path('entry', {'slug': blogPost.slug}) }}">
 {{ blogPost.title }}
</a>
{% endraw %}
{% endhighlight %}

Now refresh your browser. You'll see the title has changed to a link, and there is now a "Read more" at the bottom of your article. Click one of those and you'll see your new page!

### Showing Author Details

Want to see more details about the author of the post? Let's go to the `authorAction` in your controller. We're going to be doing something very similar to retrieving the single entry.
We'll be getting the name passed in via the URL, finding the author by name in the database. And then passing that data into the author template, as shown below:

```php
$author = $this->authorRepository->findOneByUsername($name);

if (!$author) {
    $this->addFlash('error', 'Unable to find author!');

    return $this->redirectToRoute('entries');
}

return $this->render('AppBundle:Blog:author.html.twig', [
    'author' => $author
]);
```

At the top of the method in the annotations we also want to add the service name so: `* @Route("/author/{name}")` will become: `* @Route("/author/{name}", name="author")`

With the template `./src/AppBundle/Resources/views/Blog/author.html.twig`, we won't be doing anything special, just outputting the data the Author has:

{% highlight html %}
{% raw %}
{% extends "::base.html.twig" %}

{% block title %}AppBundle:Blog:author{% endblock %}

{% block body %}
<div class="container">
    <div class="blog-header">
        <h1 class="blog-title">Author</h1>
        <p class="lead blog-description">A brief look into the author.</p>
    </div>

    <div class="row">
        <div class="col-sm-8 blog-main">
            <div class="blog-post">
                <h2 class="blog-post-title">{{ author.name|raw }}</h2>

                <p>Title: {{ author.title|raw }}</p>
                <p>Company: {{ author.company|raw }}</p>
                <p>Short Biography: {{ author.shortBio|raw }}</p>
                <ul>
                    {% if author.getPhone %}
                        <li>{{ author.phone }}</li>
                    {% endif %}
                    {% if author.getFacebook %}
                        <li>{{ author.facebook }}</li>
                    {% endif %}
                    {% if author.getTwitter %}
                        <li>{{ author.twitter }}</li>
                    {% endif %}
                    {% if author.getGithub %}
                        <li>{{ author.github }}</li>
                    {% endif %}
                </ul>

            </div>
        </div>
    </div>
</div>
{% endblock %}

{% endraw %}
{% endhighlight %}

We now need to have that author page linkable for people to access it. In: `./src/AppBundle/Resources/views/Blog/entries.html.twig` you will find:

{% highlight html %}
{% raw %}
{% if blogPost.author %}
    {{ blogPost.author.name }}
{% else %}
{% endraw %}
{% endhighlight %}

Let's make this a link as shown below:

{% highlight html %}
{% raw %}
{% if blogPost.author %}
    <a href="{{ path('author', {'name': blogPost.author.username|url_encode }) }}">
        {{ blogPost.author.name }}
    </a>
{% else %}
{% endraw %}
{% endhighlight %}

And in `./src/AppBundle/Resources/views/Blog/entry.html.twig` you will find:

{% highlight html %}
{% raw %}
<p class="blog-post-meta">{{ blogPost.updatedAt|date('F j, Y') }} by {{ blogPost.author.name }}</p>
{% endraw %}
{% endhighlight %}

Let's replace it with the following:

{% highlight html %}
{% raw %}
<p class="blog-post-meta">
    {{ blogPost.updatedAt|date('F j, Y') }} by <a href="{{ path('author', {'name': blogPost.author.username|url_encode }) }}">
        {{ blogPost.author.name }}
    </a>
</p>
{% endraw %}
{% endhighlight %}

Refresh your browser, you will see the author names have a link now.

{% include tweet_quote.html quote_text="Urray! I've just finished creating my own blog engine with Symfony and PHP!" %}

## Conclusion

Congratulations, you have just built yourself a functional blog engine from scratch with Symfony. This blog engine even enables visitors to sign up to become authors. This allows them to also contribute to your blog by posting articles of their own! Although this is just the basics of a blog, it is a strong stepping stone into making it as custom and feature filled as you wish.
