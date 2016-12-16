---
title:  Remonter les erreurs provenant d'un client (API) sur un formulaire
layout: post
tags:
    - Symfony
    - Formulaire
    - API
---

Remonter les erreurs provenant d'un client (API) sur un formulaire
------------------------------------------------------------------

Il peut être interressant voir même incontournable de remonter les erreurs emanant d'un formulaire qui fait appel à une API via un client afin de prévenir l'utilisateur.

Notre client, [reprenons un exemple déjà utilisé] (https://github.com/20uf/php-good-practice/blob/master/tests/guzzle/FacebookClient.php)

```php
    ...
    class FacebookClient extend implements ClientInterface
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
    
...

```php
    ...
    class FacebookClient implements ClientInterface
    {
       ...

    /**
         * Adds a custom error to the current form.
         * 
         * @param string $errorMessageId The id used in translation for a form message error.
         * @param string $childrenKey    The key used to extract the children form input.
         * 
         * @return void
         */
        protected function addFormError($errorMessageId, $childrenKey=null)
        {
            $errorMessage = $this->translator->trans($errorMessageId, array(),
                                                     'validators');
            $formError    = new FormError($errorMessage);
            if (isset($childrenKey))
            {
                // Add error to the children form
                $formChildrens = $this->form->all();
                $formWidget    = $formChildrens[$childrenKey];
                $formWidget->addError($formError);
            }
            else 
            {
                $this->form->addError($formError);
            }
        }
```

