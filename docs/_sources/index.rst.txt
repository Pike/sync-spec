Mozilla L10n Sync
=================

This document details how to tools should interact with repositories holding
Mozilla localization data.

Bi-directional Sync
===================

With frequent changes on the upstream repository for refactorings and
technical intervention, it's important to allow for bi-directional sync
between L10n tools and repositories. The algorithm described below is
one approach, the goals are clearly defined, though.

Goals
-----

The key goal is to allow technical interaction with the localization files
without disturbing the localization workflow.

* Refactorings don't create localization work unless required
* Migrations from legacy formats don't create localization work
* In case of conflicts, the changes made in VCS take precedence over those in the workbench.

Algorithm
---------

The following algorithm is described in terms and processes of git version control
graphs.

Initial State
^^^^^^^^^^^^^

We're starting off with a tree with the last sync on the 4th revision.

.. digraph:: DAG

   graph [
       size = "1,3";
   ];
   node [
     shape = "box"
   ];

   1 -> 2;
   2 -> 3;
   3 -> "last sync";

Then there's one or more commits directly to the repository, by other tools or
project managers.

.. digraph:: update

   graph [
       size = "1,1.5";
   ];
   node [
     shape = "box"
   ];
   "last sync" -> "master";

Updating References Phase
^^^^^^^^^^^^^^^^^^^^^^^^^

Find differences between ``last-sync`` and ``master`` for the reference locale.
Synchronise those difference back into the database, ensuring that

* removed reference strings are obsoleted in the database for all (affected) locales,
* changed reference strings are updated, without triggering localization workflows,
* new reference strings not yet added.

This step prepares for the synchronization from the tool into the repository, and
ensures that removed strings are removed from the localizations.

This step also takes changes to the project configuration into account. Resources
might be moved into the scope of a project, or the project might change scope
by adding or removing resource locations, or by changing the set of exposed
locales for a particular resource. The example in the `ProjectConfig`_ documentation
details on how the config files impact resource and locale mappings.

Tool -> Repo Phase
^^^^^^^^^^^^^^^^^^

The first direction of the sync algorithm is to extract the data from the tool to
version control. This is done by creating a commit on the  ``last sync`` branch.

.. digraph:: update

   graph [
       size = "2,1.5";
   ];
   node [
     shape = "box"
   ];
   4 -> "master";
   4 -> "last sync";

Rebase
^^^^^^

The new commit is rebased on top of ``master``, leaving a branch marker on the old
commit.

.. digraph:: update

   graph [
       size = "2,2";
   ];
   node [
     shape = "box"
   ];
   4 -> "master" -> "last sync";
   4 -> "staging";

Merge & Push
^^^^^^^^^^^^

The ``last sync`` is fast-forward merged into ``master``, and both branches are pushed.

If the push fails, for example due to new remote content, the sync is restarted.

Repo -> Tool Phase
^^^^^^^^^^^^^^^^^^

The differences between ``staging`` and ``last sync`` are pushed back to the
tool's database. This will have

* new content to translate in the reference locale,
* modifications to existing translations,
* and newly submitted translations.

Clean up
^^^^^^^^

At this point, the local ``staging`` branch is deleted.


.. _`projectconfig`: https://moz-l10n-config.readthedocs.io/en/latest/examples.html
