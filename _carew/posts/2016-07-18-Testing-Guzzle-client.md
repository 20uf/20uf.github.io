---
title:  Tester le client Guzzle dans les tests unitaires
layout: post
tags:
    - php5
    - Guzzle
    - Phake
    - Mock
---

Tester le client Guzzle dans les tests unitaires
---------------------

Guzzle est un client HTTP qui envoie des requêtes HTTP à un serveur et reçoit des réponses HTTP.

Il est possible depuis la version 6 de mocker les réponses, prenons pour contexte un client facebook.

J'utilise le framework [Phake](http://phake.readthedocs.io/en/2.1/) en complément de [PHPUnit](https://phpunit.de/).

```php

    // App/Client/FacebookClient.php
    namespace App\Client;

    use .. 

    /**
     * Class FacebookClient
     *
     * @author Michael COULLERET <michael@coulleret.pro>
     */
    class FacebookClient
    {
        ... 
        
        /**
         * {@inheritdoc}
         */
        public function getMessageFromUrl($url)
        {
            if (!preg_match('`^https?://www.facebook.com/([A-Za-z0-9]+)+/posts/([0-9]+)$`', $url, $match)) {
                throw new \Exception('Invalid URL facebook');
            }
    
            $facebookUser = $this->getUser($match[1]);
            $posts = $this->getPosts($match[1]);
    
            $messageId = sprintf('%s_%s', $facebookUser['id'], $match[2]);
    
            foreach ($posts['data'] as $post) {
                if ($post['id'] === $messageId) {
                    $post['name'] = $facebookUser['name'];
                    $post['avatar'] = sprintf('http://graph.facebook.com/%s/picture', $match[1]);
    
                    return $post;
                }
            }
    
            throw new NotFoundHttpException(sprintf('Message id %s do not found', $messageId));
        }
        
        ...
    }

```

L'objectif est de simuler les appels dans une file d'attente:

```php

    // tests/App/Client/FacebookClient.php
    namespace Tests\Client;

    /**
     * Class FacebookClientTest
     *
     * @author Michael COULLERET <michael@coulleret.pro>
     */
    class FacebookClientTest extends \PHPUnit_Framework_TestCase
    {
        /**
         * @test
         */
        public function shouldGetMessageFromUrl()
        {
            $guzzleMock = new MockHandler([
                new Response(200, [], $this->mockJsonOauth),
                new Response(200, [], $this->mockJsonUser),
                new Response(200, [], $this->mockJsonPosts),
            ]);
    
            $guzzleClient = new Client(['handler' => HandlerStack::create($guzzleMock)]);
    
            $clientFacebook = \Phake::partialMock(FacebookClient::class, $this->config, $guzzleClient, $this->logger);
    
            $result = $clientFacebook->getMessageFromUrl($this->mockUrl);
    
            $mockToArray = json_decode($this->mockJsonPosts, true);
            $mockVerify = $mockToArray['data'][2];
            $mockVerify['name'] = 'Barack Obama';
            $mockVerify['avatar'] = 'http://graph.facebook.com/barackobama/picture';
    
            $this->assertEquals($result, $mockVerify);
        }
        
        ...
        
    }
```

La classe MockHandler() permet de créer une liste d'attente de réponse, dans notre contexte la première réponse se charge de renvoyer l'access token de facebook, la seconde l'entité utilisateur et la troisième une liste de posts de l'utilisateur.
Nous passons notre file d'attente au client Guzzle qui se comportera dans de vrais conditions, efficace pour avoir des tests intègres.

Le tour est joué !

Le code exemple est disponible à cette adresse : https://github.com/20uf/php-good-practice/tree/master/tests/guzzle

Documentation complète sur le site officiel: http://docs.guzzlephp.org/en/latest/testing.html