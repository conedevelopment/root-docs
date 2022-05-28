---
title: "Core Concepts"
github: "https://github.com/conedevelopment/root-docs/blob/master/core-concepts.md"
order: 1
---

## What is Root?

Root is an admin package for Laravel applications. It offers a simple model management with a robust resource API and a modern and fast UI.

## Scope and Scale

Root is a quite simple, drop-in admin package. It means, it requires less configuration when installing, so probably it's easier to migrate between versions, or migrate between projects.

However, it comes with a price: Root is scoped to small, medium-sized projects: e-commerce or generic CMS projects.

## Comparing to other packages, solutions

Root is heavily inspired by [Symfony EasyAdmin](https://symfony.com/bundles/EasyAdminBundle/current/index.html) and [Laravel Nova](https://nova.laravel.com/).

> Please note, that we do not hold a Laravel Nova license and never seen its source code. Any code we've seen regarding Nova is in its docs or publicly available resources.

Root is targeting smaller applications and offers simpler and faster integration into your running or fresh application. But for very large applications you may consider using Nova. Nova has a big community and first-party support since it's maintained by the Laravel team. Also, it has some features (Queueable Actions, Vapor and Scout support for example) that we don't and probably won't support in the future.
