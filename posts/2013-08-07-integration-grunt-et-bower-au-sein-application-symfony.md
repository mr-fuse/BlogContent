# Intégration Grunt et Bower au sein d'une application Symfony

## Définition des outils

> Test

### Grunt

Développé en Javascript et executé par NodeJS, **[Grunt][grunt]** est utilisé par un large spectre de développeur, et présente de réels atouts par rapport à **Assetic**.  
A première vue, le fait qu'il n'y ai pas d'intégration direct à Symfony2 peut apparaitre comme un désavantage, mais ne pas coupler l'application Symfony2 à la configuration de votre gestionnaire de ressources vous permet une plus grande souplesse.

> Pourquoi utiliser un gestionnaire de tâche ?  
> En un mot : L'automatisation. Moins vous avez de travail quand vous executez des tâches répétitives comme la **minification**, la **compilation**, les **tests unitaires**, le **linting**, etc, plus votre travail deviendra simple. Après l'avoir configuré, le gestionnaire de tâche peut faire la plupart du travail trivial pour vous et votre équipe sans aucun effort.

### Bower

Développé et executé aussi par le couple Javascript/NodeJS, **[Bower][bower]** est un gestionnaire de dépendance pour le développement **front-end**.  
Pour faire une comparaison rapide, Bower est l'équivalent de Composer pour les librairies Javascript & CSS.

## Cas d'utilisation, le besoin et la réponse

Chez Wozbe,  
Nous avons pris l'habitude d'utiliser le couple **Bower et Grunt** pour gérer les librairies **front-end** tels que [jQuery][jquery], [YUI][yui], [Bootstrap][bootstrap], [Chosen][chosen], [LESS][less]...

> Certaines parties incluant une structure spécifique peuvent être contestable. Il s'agit d'essais sur nos projets internes afin d'en mesurer l'usage.

### Structure de fichier

Pour commencer, voyons la structure de nos ressources.  

~~~~~
# Les bundles
# Standard symfony
# Contient les sources publiques de votre bundle
src/Acme/DemoBundle/Resources/public/images
src/Acme/DemoBundle/Resources/public/js
src/Acme/DemoBundle/Resources/public/less

# L'application
# Non standard symfony
# Contient les sources publiques de votre application
app/Resources/public/fonts
app/Resources/public/images
app/Resources/public/js
app/Resources/public/less
~~~~~

#### Les bundles
Il est fort probable que la structure de vos bundles corresponde à la structure **recommandée par Symfony2**. Ainsi vous utilisez le répertoire **src/Acme/DemoBundle/Resources/public/** pour mettre tous les fichiers qui devront être disponibles pour vos clients.

De ce fait, vous utilisez la commande *app/console assets:install* pour déployer ces fichiers dans le répertoire **web/**.  
Ce déploiement peut-être réalisé par une copie de fichier ou par la mise en place de lien symbolique avec l'option **--symlink**.

Cette opération va donc créer un répertoire **web/bundles/acmedemo/** contenant les fichiers du répertoire **src/Acme/DemoBundle/Resources/public/**

#### L'application
Pour la structure des ressources de l'application, nous prenons une liberté vis-à-vis du standard Symfony.  
La documentation indique que pour surcharger les templates d'un bundle nous devons utiliser le répertoire **app/Resources/AcmeDemoBundle/views**.  
A ce titre, nous pensons qu'il faut utiliser le répertoire **app/Resources/public** pour gérer les fichiers publics de l'application.

Une de nos tâches [Grunt][grunt] consistera à mettre dans le répertoire **web/bundles/app/** le contenu de **app/Resources/public**, ainsi nous traitons les ressources de l'application de la même manière que nous le faisons avec un bundle.

### Ce que l'on attends
Nous avons décrit la structure des fichiers sources. Sur l'utilisation des bundles il n'y a rien de différent par rapport à vos habitudes. On change par contre la manière de procéder pour les données communes à votre application.

Voici la structure de vos ressources que l'on aura après exportation via Grunt.

~~~~~
# Les sources sont copiées dans web/bundles/
# On retrouve les bundles et l'application
web/bundles/acmedemo/images
web/bundles/acmedemo/js
web/bundles/acmedemo/less
web/bundles/app/fonts
web/bundles/app/images
web/bundles/app/js
web/bundles/app/less
~~~~~

Au sein de notre projet,  
Nous allons appliquer différents traitements sur nos fichiers sources, comme la compilation des fichiers LESS en CSS, minification et/ou obfuscation des sources Javascript.

Nous définissons un répertoire spécifique aux fichiers modifiés : **web/built/**

~~~~~
# Les fichiers modifiés sont dans web/built/
web/built/acmedemo/js
web/built/acmedemo/css
web/built/app/js
web/built/app/css
~~~~~

Vous remarquerez que les répertoires **web/built/*/images/** et **web/built/*/fonts/** ne sont pas définis, les fichiers associés n'ayant reçu aucune modification. Ils restent dans **web/bundles/**

## Cas d'usage : Symfony avec Bootstrap, jQuery et FontAwesome

Voici le cas d'usage d'une intégration **Bootstrap, jQuery et FontAwesome** au sein d'une application Symfony2 grâce à Bower & Grunt.

### Définition des sources LESS

Passons à l'écriture de nos fichiers LESS.  
Il s'agit ici d'un choix spécifique pour notre application, où l'on définit deux styles, un généraliste et un spécifique.
~~~~~
# app/Resources/public/less/wozbe.less

@import "web/vendor/bootstrap/less/bootstrap.less";
@import "web/vendor/bootstrap/less/responsive.less";
@import "web/vendor/bootstrap/less/variables.less";

@import "web/vendor/font-awesome/less/font-awesome.less";

@FontAwesomePath:   "../../../bundles/app/fonts/awesome/";
@iconSpritePath: '../../../bundles/app/images/glyphicons-halflings.png';
@iconWhiteSpritePath: '../../../bundles/app/images/glyphicons-halflings.png';

body {
  background: @blue;
}
~~~~~

~~~~~
# src/Acme/DemoBundle/Resources/public/less/index.less

.blog-index {
  background: @green;
}
~~~~~

L'exportation de ces fichiers se fera dans le répertoire **web/bundles/**, et la compilation dans **web/built/**
~~~~~
# Les fichiers exportés sont dans web/bundles/
web/bundles/acmedemo/less/index.less
web/bundles/app/less/wozbe.less

# Les fichiers modifiés sont dans web/built/
web/built/acmedemo/css/index.css
web/built/app/css/wozbe.css
~~~~~

Pour inclure ces fichiers à vos pages, vous pouvez utiliser la fonction Twig **assets**
~~~~~
<link rel="stylesheet" href="{{ asset('built/app/css/wozbe.css') }}" type="text/css"/>
<link rel="stylesheet" href="{{ asset('built/acmedemo/css/index.css') }}" type="text/css"/>
~~~~~

### Définition des sources JavaScript

Passons à l'écriture de nos fichiers JavaScript.  

~~~~~
# app/Resources/public/js/wozbe.js

// Do code JavaScript qui fait quelque chose de sympa
(function () {
  'use strict';
  
  var onHashChange = function() { 
    jQuery('#element').css('background', 'blue');
  };
  
  window.addEventListener('hashchange', onHashChange);
}());
~~~~~

L'exportation de ce fichier se fera dans le répertoire **web/bundles/app/**, et la compilation dans **web/built/app/**
~~~~~
# Les fichiers exportés sont dans web/bundles/
web/bundles/app/js/wozbe.js

# Les fichiers modifiés sont dans web/built/
web/built/app/js/wozbe.js
~~~~~

Pour inclure ces fichiers à vos pages, vous pouvez utiliser la fonction Twig **assets**
~~~~~
<script src="{{ asset('built/app/js/wozbe.js') }}"></script>
~~~~~

### Configuration et Utilisation de Bower

Comme nous l'avons vu au début de l'article, Bower est un gestionnaire de dépendance **front-end**.

~~~~~
# Installation de Bower
npm -g install bower
~~~~~

La configuration de Bower passe par un fichier caché **.bowerrc**. Vous pourrez configurer le répertoire de destination des librairies, ainsi que le fichier contenant la liste de vos dépendances.
~~~~~
# .bowerrc
{
  "directory": "web/vendor",
  "json": "bower.json"
}
~~~~~
Comme vous pouvez le voir, nous utilisons le répertoire **web/vendor** comme lieux d'accueil des librairies front-end.  
L'avantage, l'ensemble des librairies sera disponible directement, et l'inconvéniant, il vous faudra utiliser des librairies de confiance.

> Si vous souhaitez aller plus loin. Vous pouvez utiliser un répertoire non disponible via HTTP, comme **vendor-front/**

Pour définir les dépendances, nous allons éditer un fichier **bower.json** qui a le même rôle que le fichier **composer.json**
~~~~~
# bower.json
{
  "name": "wozbe",
  "version": "0.1.0",
  "main": "bower.json",
  "ignore": [
    "**/.*",
    "node_modules",
    "components"
  ],
  "dependencies": {
    "jquery": "1.8.0",
    "bootstrap": "2.3.2",
    "font-awesome": "3.2.1"
  }
}
~~~~~

Le déployement des librairies se fera simplement avec **bower install**.

### Configuration et Utilisation de Grunt

Passons enfin à l'essence de l'article, la configuration et l'utilisation de Grunt.

~~~~~
# Installation de Grunt CLI
npm install -g grunt-cli
~~~~~

Le rôle de [GruntCLI][grunt_cli] est d'exécuter la version de Grunt correspondant au fichier de description **Gruntfile.js**.  
Ainsi, vous n'avez qu'un utilitaire d'exécution grunt sur votre système, et différentes versions de Grunt pour chacun de vos projets peuvent coexister.

Grâce au fichier **package.json**, nous allons définir les dépendances NodeJS et Grunt qui seront installées par [NPM][npm]

~~~~~
# package.json
{
  "name": "wozbe",
  "version": "0.1.0",
  "author": "Thomas Tourlourat <thomas@tourlourat.com>",
  "private": true,
  "dependencies": {
    "grunt": "~0.4.1",
    "grunt-contrib-less": "~0.6.1",
    "grunt-contrib-concat": "~0.3.0",
    "grunt-contrib-watch": "~0.4.4",
    "grunt-symlink": "~0.4.0",
    "grunt-contrib-jshint": "~0.6.0",
    "grunt-contrib-uglify": "~0.2.2"
  }
}
~~~~~

La récupération des dépendances se fera via **npm install**.  
Pour ceux qui n'ont jamais utilisé NodeJS, vous verrez que les modules sont installés dans le répertoire **node_modules/**.

La configuration de **Gruntfile.js** sera ici simplifiée pour les besoins de l'article. Vous trouverez la version complète de ce fichier pour wozbe.com sur [Github][gruntfile_github].

~~~~~
# Gruntfile.js
module.exports = function(grunt) {
  grunt.loadNpmTasks('grunt-symlink');
  grunt.loadNpmTasks('grunt-contrib-less');
  grunt.loadNpmTasks('grunt-contrib-concat');
  grunt.loadNpmTasks('grunt-contrib-watch');
  grunt.loadNpmTasks('grunt-contrib-uglify');

  // Création du répertoire d'image pour l'application s'il n'existe pas.
  grunt.file.mkdir('app/Resources/public/images/');

  var filesLess = {};

  // Gestion des fichiers LESS
  // Les sources des fichiers LESS sont situées : web/bundles/[bundle]/less/
  // La destination souhaitées des fichiers compilés CSS : web/built/[bundle]/css/ et web/built/app/css/
  var mappingFileLess = grunt.file.expandMapping(
    ['*/less/*.less', '*/less/*/*.less'], 
    'web/built/', {
      cwd: 'web/bundles/',
      rename: function(dest, matchedSrcPath, options) {
        return dest + matchedSrcPath.replace(/less/g, 'css');
      }
    });

  // Construction de l'objet "filesLess"
  // Les propriétés de l'objet font réference à la destination souhaitée (CSS), et leurs valeurs à la source des fichiers (LESS)
  grunt.util._.each(mappingFileLess, function(value) {
    // Why value.src is an array ??
    filesLess[value.dest] = value.src[0];
  });
  
  // Configuration du projet
  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),
    
    // Définition de la tache 'less'
    // https://github.com/gruntjs/grunt-contrib-less
    less: {
      bundles: {
        files: filesLess
      }
    },
    // Définition de la tache 'symlink'
    // https://github.com/gruntjs/grunt-contrib-symlink
    symlink: {
      // app/Resources/public/ doit être disponible via web/bundles/app/
      app: {
        dest: 'web/bundles/app',
        relativeSrc: '../../app/Resources/public/',
        options: {type: 'dir'}
      },
      // Gestion des glyphicons
      bootstrap_glyphicons_white: {
        dest: 'app/Resources/public/images/glyphicons-halflings-white.png',
        relativeSrc: '../../../../web/vendor/bootstrap/img/glyphicons-halflings-white.png',
        options: {type: 'file'}
      },
      // Gestion des glyphicons
      bootstrap_glyphicons: {
        dest: 'app/Resources/public/images/glyphicons-halflings.png',
        relativeSrc: '../../../../web/vendor/bootstrap/img/glyphicons-halflings.png',
        options: {type: 'file'}
      },
      // Gestion de FontAwesome
      font_awesome: {
        dest: 'app/Resources/public/fonts/awesome',
        relativeSrc: '../../../../web/vendor/font-awesome/font/',
        options: {type: 'dir'}
      }
    },
    concat: {
      dist: {
        src: [
          'web/vendor/jquery/jquery.js',
          'web/vendor/bootstrap/js/bootstrap-transition.js',
          'web/vendor/bootstrap/js/bootstrap-alert.js',
          'web/vendor/bootstrap/js/bootstrap-modal.js',
          'web/vendor/bootstrap/js/bootstrap-dropdown.js',
          'web/vendor/bootstrap/js/bootstrap-scrollspy.js',
          'web/vendor/bootstrap/js/bootstrap-tab.js',
          'web/vendor/bootstrap/js/bootstrap-tooltip.js',
          'web/vendor/bootstrap/js/bootstrap-popover.js',
          'web/vendor/bootstrap/js/bootstrap-button.js',
          'web/vendor/bootstrap/js/bootstrap-collapse.js',
          'web/vendor/bootstrap/js/bootstrap-carousel.js',
          'web/vendor/bootstrap/js/bootstrap-typeahead.js',
          'web/vendor/bootstrap/js/bootstrap-affix.js',
          'web/bundles/app/js/wozbe.js'
        ],
        dest: 'web/built/app/js/wozbe.js'
      }
    },
    // Lorsque l'on modifie des fichiers LESS, il faut relancer la tache 'css'
    // Lorsque l'on modifie des fichiers JS, il faut relancer la tache 'javascript'
    watch: {
      css: {
        files: ['web/bundles/*/less/*.less'],
        tasks: ['css']
      },
      javascript: {
        files: ['web/bundles/app/js/*.js'],
        tasks: ['javascript']
      }
    },
    uglify: {
      dist: {
        files: {
          'web/built/app/js/wozbe.min.js': ['web/built/app/js/wozbe.js']
        }
      }
    }
  });

  // Default task(s).
  grunt.registerTask('default', ['css', 'javascript']);
  grunt.registerTask('css', ['less']);
  grunt.registerTask('javascript', ['concat', 'uglify']);

};
~~~~~

Ce fichier de configuration [Grunt][grunt] permet de réaliser ce que nous avons évoqué au cours de l'article.  

### Résumé des commandes utiles

Pour l'extraction des ressources de vos bundles et de vos applications.
~~~~~
php app/console assets:install --symlink && grunt symlink
~~~~~

Pour Compiler, minifier, etc.. 
~~~~~
grunt
~~~~~

L'option **watch** vous permet d'exécuter toutes les tâches Grunt nécessaires à votre applications à chaque modification d'un fichier source.
~~~~~
grunt watch
~~~~~

Si vous avez des commentaires à faire sur cette approche, n'hésiter surtout pas.

[jquery]: http://jquery.com/  "jQuery"
[yui]: http://yuilibrary.com/  "YUI"
[bootstrap]: http://getbootstrap.com/  "Twitter Bootstrap"
[chosen]: http://harvesthq.github.io/chosen/  "Chosen"
[less]: http://lesscss.org/  "LESScss"
[grunt]: http://gruntjs.com/  "GruntJS"
[bower]: https://github.com/bower/bower  "Bower"
[npm]: https://npmjs.org/  "NPM"
[gruntfile_github]: https://github.com/wozbe/wozbe.com/blob/master/Gruntfile.js  "Gruntfile Github"
[grunt_cli]: https://github.com/gruntjs/grunt-cli  "Grunt CLI"
