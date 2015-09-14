---
layout:     post-no-pic
title:      "Reporting PHPUnit results from HarborMaster"
subtitle:   "Basics of reporting results of a PHPUnit from a remote build server"
date:       2015-09-13 02:41:00
author:     "Thomas Barthelemy"
tags:       [phabricator, git]
---

In a previous post ([Running PHPUnit Build with HarborMaster and DryDock](http://thomas-barthelemy.github.io/2015/09/11/phpunit-phabricator-drydock-harbormaster/))
we discussed how to use Docker, HarborMaster and DryDock to run custom build operations,
but the details won't show up under `Unit` in your **Diff** like it would do if
ran by *Arcanist*.

# Conduit

The savior here gonna be **Conduit** and more specifically [harbormaster.sendmessage](https://secure.phabricator.com/conduit/method/harbormaster.sendmessage/)
which allows to send **Unit** or **Lint** results through API calls.

# What you need

## buildTargetPHID

This is available as one of the variable provided by **HarborMaster**:

    $target.phid

## type

The type of result, that will be one of the following: `pass`, `fail` or `work`.
Those values are detailed in the [Conduit API Documentation](https://secure.phabricator.com/conduit/method/harbormaster.sendmessage/)

It will be necessary to understand our Unit results to give the right value.

## unit

This is the core part, we have to get the right JSON format for our results,
here is a simple possibility using [jq](https://stedolan.github.io/jq/) (`apt-get install jq`):

    # Running PHPUnit and saving results as JSON
    phpunit --log-json unitlog.json

    # Formatting result
     cat unitlog.json | jq -s '
     map(select(.status)) |
        {
            buildTargetPHID:"$target.phid",
            type: (
                if (map(select(.status != "pass"))|length) == 0
                then "pass"
                else "fail"
                end
            ),
            unit:map({
                name: .test,
                result: .status,
                duration: .time
            })
        }' > result.json

    # Calling conduit
    cat result.json | arc call-conduit harbormaster.sendmessage

Uh... Ok this piece of `jq` was written in the middle of the night and would deserve
some improvement. Note that I did not use functions like `any` as not all ubuntu `jq` packages
have those available.

Just a little detail on "what's kind of sorcery is this" in case you plan on changing it:

 - `jq -s`: Interpret the JSON as a whole rather than a set of independent json objects
 - `map(select(.status))`: Select only the objects with a status field (actual test result)
 - `{ ... }`: We create our expected JSON
 - `map(select(.status != "pass"))|length`: Will give the number of status that are not `pass`
 - `map({name: .test, result: .status, duration: .time}`: We create a new array with objects having only the fields we need

# What's next?

This result does not really care of the different possibilities like skipped test, this could be easily added.
