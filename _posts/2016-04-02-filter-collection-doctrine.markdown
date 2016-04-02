---
layout:     post-no-pic
title:      "Filtering doctrine collections"
subtitle:   "How to filter collections on the database side in a doctrine entity"
date:       2016-04-02 18:33:00
author:     "Thomas Barthelemy"
tags:       [php]
---

I recently ran into the following situation as defined:

 - An entity A with a `OneToMany` relation to entity B
 - FOS REST Bundle serializing entity and returning it in multiple API endpoints
 - A entity has a property `hidden`

I wanted to see what was the simplest and most efficient way for all those APIs
and for anywhere using `A->getBs()` to retrieve the list of B related to A
that have `hidden = false`.

Basically I needed something that would look like:

    @OneToMany
    @Where(!hidden)

Which is obviously not a feature.

1st idea
-------

One possible way is to updated everywhere the collection is used and do the filtering.
That means a lot of things to change, which is not really maintainable, and it
does not happen on the database level so not really great performances.

2sd idea
-------

The second though was to manually query what was needed wherever it was needed.
This could have worked but a few things got in the way.

 - It's still not maintainable, it's a lot of changes in a lot of different places
 - So far the FOS REST bundle was serializing things automatically which is very convenient, definitely didn't want to change that

 Criteria
 -------

 The first step towards the light was using criteria, changing the getter definition as follows:

     /**
      * Get Addresses
      *
      * @return B[]
      */
     public function getBs()
     {
         $criteria = Criteria::create();
         $criteria->where(Criteria::expr()->eq('hidden', false));

         return $this->b->matching($criteria);
    }

Criteria are quite amazing as it's a neat way to filter collections and, as said in the doctrine documentation:

    Collections have a filtering API that allows to slice parts of data from a collection.
    If the collection has not been loaded from the database yet,
    the filtering API can work on the SQL level to make optimized access to large collections.

A good thing to note, is that the `QueryBuilder` can now takes Criteria as well.

After that I noted that the serialization was still going directly to the property instead of using the getter.
The final key was the `Accessor` attribute that helps specify which getter and setter to use:

    /**
    * @var B[]
    *
    * @ORM\OneToMany(...)
    * @Accessor(getter="getBs")
    *
    */
    private $b;

And that's it, Working!
