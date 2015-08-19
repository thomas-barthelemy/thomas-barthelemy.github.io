---
layout:     post-no-pic
title:      "Displaying a simple Task tree in Phabricator"
subtitle:   "How to display the children of a task in a visual tree in Phabricator's Maniphest"
date:       2015-09-19 12:01:00
author:     "Thomas Barthelemy"
tags:       [phabricator]
---

We recently migrated our workflow to [Phabricator](http://phabricator.org/)
and while it supports parent/children relations for tasks, it stills display those
tasks as a list in Maniphest.
The main reason is that it allows a taslto have multiple parents which is complicated to display.

We've come up with a simple first solution using a custom field:

```php
<?php
final class TaskTreeCustomField extends ManiphestCustomField {

    public function getFieldKey() {
        return 'webridge:tasktree';
    }

    public function shouldAppearInPropertyView() {
        return true;
    }

    public function renderPropertyViewLabel() {
        return pht('Dependency Graph');
    }

    public function renderPropertyViewValue(array $handles) {
        $task = $this->getObject();

        $edge_type = ManiphestTaskDependsOnTaskEdgeType::EDGECONST;

        $graph = id(new PhabricatorEdgeGraph())
            ->setEdgeType($edge_type)
            ->addNodes(
                array(
                    '<seed>' => array($task->getPHID()),
                ))
            ->loadGraph();

        $nodes = $graph->getNodes();
        unset($nodes['<seed>']);

        if (count($nodes) == 1) {
            return null;
        }

        $phids = array_keys($nodes);
        $handles = id(new PhabricatorHandleQuery())
            ->setViewer($this->getViewer())
            ->withPHIDs($phids)
            ->execute();

        return $this->drawNodes($task->getPHID(), $nodes, $handles, true);
    }

    private function drawNodes($phid, $nodes, $handles, $is_top = false) {
        $content = array();
        if (!$is_top) {
            $content[] = phutil_tag('li', array(), $handles[$phid]->renderLink());
        }

        foreach ($nodes[$phid] as $other) {
            $content[] = phutil_tag(
                'li',
                array(
                    'style' => $is_top ? null : 'padding-left: 16px;',
                ),
                $this->drawNodes($other, $nodes, $handles));
        }

        return phutil_tag('ul', array(), $content);
    }

}
```

This will simply flatten the graph (which can result in duplicated entry to allow a simple tree display).

To use this custom field I would recommand to read the **Advanced Custom Field** part of the [Phabricator Documentation](https://secure.phabricator.com/book/phabricator/article/custom_fields/).
The quick summary would be to put this php file in the `src/extensions/` folder of Phabricator... and that's it.