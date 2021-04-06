---
layout: single
title: Symfony 4 and FULLTEXT search
excerpt: "FULLTEXT search in symfony"
date: 2019-06-27
classes: wide
header:
    teaser: /assets/images/fulltext.gif
    teaser_home_page: true
categories:
  - Symfony
tags:
  - Symfony
---


In [this](https://github.com/beberlei/DoctrineExtensions) repository you can find some MySQL functions that you can add to your Symfony app, like in this case ‘MATCH…AGAINST’. For that, first of all create some folders inside “src”:

```bash
    mkdir -p Extensions/Doctrine
```

Now, create a new class called ‘MatchAgainst.php’ with this content:

```php
<?php

namespace App\Extensions\Doctrine;

use Doctrine\ORM\Query\AST\Functions\FunctionNode;
use Doctrine\ORM\Query\Lexer;

class MatchAgainst extends FunctionNode
{
    /** @var array list of \Doctrine\ORM\Query\AST\PathExpression */
    protected $pathExp = null;
    /** @var string */
    protected $against = null;
    /** @var bool */
    protected $booleanMode = false;
    /** @var bool */
    protected $queryExpansion = false;
    public function parse(\Doctrine\ORM\Query\Parser $parser)
    {
        // match
        $parser->match(Lexer::T_IDENTIFIER);
        $parser->match(Lexer::T_OPEN_PARENTHESIS);
        // first Path Expression is mandatory
        $this->pathExp = [];
        $this->pathExp[] = $parser->StateFieldPathExpression();
        // Subsequent Path Expressions are optional
        $lexer = $parser->getLexer();
        while ($lexer->isNextToken(Lexer::T_COMMA)) {
            $parser->match(Lexer::T_COMMA);
            $this->pathExp[] = $parser->StateFieldPathExpression();
        }
        $parser->match(Lexer::T_CLOSE_PARENTHESIS);
        // against
        if (strtolower($lexer->lookahead['value']) !== 'against') {
            $parser->syntaxError('against');
        }
        $parser->match(Lexer::T_IDENTIFIER);
        $parser->match(Lexer::T_OPEN_PARENTHESIS);
        $this->against = $parser->StringPrimary();
        if (strtolower($lexer->lookahead['value']) === 'boolean') {
            $parser->match(Lexer::T_IDENTIFIER);
            $this->booleanMode = true;
        }
        if (strtolower($lexer->lookahead['value']) === 'expand') {
            $parser->match(Lexer::T_IDENTIFIER);
            $this->queryExpansion = true;
        }
        $parser->match(Lexer::T_CLOSE_PARENTHESIS);
    }
    public function getSql(\Doctrine\ORM\Query\SqlWalker $walker)
    {
        $fields = [];
        foreach ($this->pathExp as $pathExp) {
            $fields[] = $pathExp->dispatch($walker);
        }
        $against = $walker->walkStringPrimary($this->against)
            . ($this->booleanMode ? ' IN BOOLEAN MODE' : '')
            . ($this->queryExpansion ? ' WITH QUERY EXPANSION' : '');
        return sprintf('MATCH (%s) AGAINST (%s)', implode(', ', $fields), $against);
    }
}
```

Now it’s time to register the new function. To do that, edit your `config/packages/doctrine.yaml` like this:
```yml
orm:
    ...
    dql:
        string_functions:
            MATCH_AGAINST: App\Extensions\Doctrine\MatchAgainst
```

If using doctrine annotation it’s better to annotate the entity class:
```php
@ORM\Table(name="table_name", indexes={@ORM\Index(columns={"title", "description", "author"}, flags={"fulltext"})})
```

instead of using MySql ALTER TABLE query you have to:
```bash
bin/console doctrine:migrations:diff
bin/console doctrine:migrations:migrate
```

With this, the index of yout table will be created.
Otherwise the fulltext index will be removed, next time you update your table with doctrine:migrations.

Everything is ready. Now, in your controller (or repository), you can use this function like this:
```php
/**
 * @Route("/", name="amp_index", methods={"GET"})
 *
 * @param Request            $request
 *
 *
 * @param PaginatorInterface $paginator
 *
 * @return Response
 */
public function index(Request $request, PaginatorInterface $paginator): Response
{
    /** @var EntityManager $em */
    $em = $this->getDoctrine()->getManager();

    /** @var QueryBuilder $queryBuilder */
    $queryBuilder = $em->getRepository(Amp::class)->createQueryBuilder('a');

    $filter = $request->query->get('filter');
    if ($filter) {
        $queryBuilder->where('MATCH_AGAINST(a.clasificacion, a.expediente, a.fecha, a.observaciones, a.signatura) AGAINST(:searchterm boolean)>0')
                     ->setParameter('searchterm', $filter);
    }

    $query = $queryBuilder->getQuery();

    $amps = $paginator->paginate(
        $query, /* query NOT result */
        $request->query->getInt('page', 1)/*page number*/,
        $request->query->getInt('limit', 10)/*limit per page*/
    );



    return $this->render(
        'amp/index.html.twig',
        [
            'amps' => $amps,
        ]
    );
}
```

Note a ‘boolean’ type in MATCH_AGAINST witch allows to use ‘+’, ’-’ and ‘~’ in our querys. More info here:

https://dev.mysql.com/doc/refman/5.5/en/fulltext-search.html