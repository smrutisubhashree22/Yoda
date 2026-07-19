# Yoda

A simplified Project Mangement awebsite built with Remix, created as a personal project to sharpen my skills. It's not meant to be a production-ready Jira replacement — it's still evolving, with more features and refinements on the way.

I drew inspiration from the real [Jira](https://www.atlassian.com/es/software/jira) and from [Trello](https://trello.com/), and [Ivor's Jira clone](https://github.com/oldboyxx/jira_clone) was a big influence on the visual design (honestly, I prefer his take over the original). That said, everything here — fonts and icons aside — was built from scratch by me.



## Table of Contents

1. [Setup](#setup)
   - [Install](#install)
   - [Run](#run)
   - [Build](#build)
   - [Test](#test)
2. [Overview](#overview)
   - [Login](#login)
   - [Projects List](#projects-list)
   - [Project Space](#project-space)
3. [Project Structure](#project-structure)
   - [Domain](#domain)
   - [Infrastructure](#infrastructure)
   - [App](#app)
4. [Conventions](#conventions)
5. [Technologies](#technologies)
6. [Goals](#goals)

## Setup

### Install

Setting up the app is just like any other Remix project. Start by cloning the repo:

```bash
git clone https://github.com/daniserrano7/jira-clone.git
```

Then move into the project folder and install dependencies:

```bash
cd jira-clone
npm install
```

The app relies on a SQLite database, so you'll need to set the `DATABASE_URL` environment variable. A `.sample.env` file is included as a template — copy it into your own `.env` and fill in the values. Once that's set, create the database with Prisma:

```bash
npx prisma db push
```

The database file will be created at the path defined in `DATABASE_URL`. To seed it with some starter data, run:

```bash
npx prisma db seed
```

### Run

Start the app in dev mode with:

```bash
npm run dev
```

This also runs the Tailwind CSS compiler in watch mode alongside the dev server.

### Build

To create a production build:

```bash
npm run build
```

This runs the Tailwind compiler in production mode too. Once built, start the app with:

```bash
npm run start
```

### Test

Run the test suite with:

```bash
npm run test
```

Or run ESLint, TypeScript checks, and Vitest all together:

```bash
npm run test-all
```

## Overview

The app recreates a Jira-style workspace: create projects, assign users, and manage issues. Data is stored in a SQLite database — locally in development, and on Fly's persistent storage in the live demo. Thanks to Remix's SSR approach, almost everything is rendered server-side, with client-side state used only for session data and issue search.

The app has three main sections: login, projects list, and project space.

### Login

The login screen is the first thing you'll see. Pick a user from a simple select input to log in — this creates a cookie session used to authenticate you server-side. Without it, you'll be redirected back to the login page. You can log out anytime via the avatar menu in the top-right corner, which clears the session and sends you back to login.



### Projects List

This view lists the projects the logged-in user has access to, and lets you create new projects and assign users to them.



### Project Space

This is the heart of the app: the project board, showing all its issues. There are additional tabs for analytics (not yet implemented) and backlog (purely for UI purposes — it won't be built out). A couple of links intentionally trigger 404 and 500 errors, just to show off Remix's error handling.

From the board, you can create, edit, and delete issues; update their properties and category; add comments; assign them to users; and drag and drop issues between categories. Every change streams to the server via Server-Sent Events, so the board updates in real time — try opening two tabs side by side to see it in action.



Clicking into an issue opens an editing panel with all its details. It's designed to be simple and intuitive, with server-side validation built in. Despite living on its own URL route, it renders as a modal with a smooth transition.

<
## Technologies

Built with [React](https://reactjs.org/) (ES6 + hooks), [TypeScript](https://www.typescriptlang.org/), and [Remix](https://remix.run/) for SSR. UI components use [Radix](https://www.radix-ui.com/) for accessibility, styled with [Tailwind CSS](https://tailwindcss.com/). Remix's SSR-first approach keeps client-side state to a minimum.

Data lives in a [SQLite](https://www.sqlite.org) database, managed through the [Prisma](https://www.prisma.io/) ORM. Testing is handled with [Vitest](https://vitest.dev/) — though since I moved to an entity-based approach, domain-level tests are no longer needed. Testing routing and HTTP requests is next on the list.

Linting is done with [ESLint](https://eslint.org/) (fairly relaxed rules), formatting with [Prettier](https://prettier.io/), and the app is deployed on [Fly.io](https://fly.io/).

## Project Structure

Following [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) principles, the source is split into three layers: domain, infrastructure, and app (UI). The root also has a `public` folder for static assets like images, fonts, and build output.

### Domain

The domain folder holds all entities and value objects. Entities are modeled mainly as TypeScript interfaces. Each entity folder typically contains:

- `[entity].ts` — interfaces, value objects, and the factory function
- `[entity].mock.ts` — mock data used for seeding and examples
- `index.ts` — re-exports everything else

This keeps coupling low: entities depend only on themselves or lower-level entities. The hierarchy looks like this:

```
users
projects
└───categories
    └───issues
        └───user
        └───comments
            └───user
```

> **Note:** this diagram only shows relationships between entities — non-relevant attributes are omitted.

### Infrastructure

This layer covers anything that supports the app's logic but isn't part of the UI and isn't tied to a specific framework — currently just the database layer, though it could expand to things like local storage later.

Cookie sessions live outside this layer since they're Remix-specific. Store logic and related components also live in the app folder, since they're tightly coupled to the implementation.

### App

Everything Remix-related — UI and backend alike:

- **Components** — shared, generic components used across the app
- **Routes** — Remix routes and their SSR logic
- **Session-storage** — cookie session handling for auth and theme
- **Store** — Context API stores for sharing user and theme state
- **Styles** — Tailwind global styles and font definitions
- **UI** — a hierarchical folder structure that mirrors the UI composition itself, inspired by Robert Martin's [Screaming Architecture](http://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html), so the folder layout makes the app's structure easy to navigate

## Conventions

Files and folders use [kebab-case](https://en.wiktionary.org/wiki/kebab_case) for better readability:

> _e.g._ `issue-card`

React components use standard [PascalCase](https://en.wiktionary.org/wiki/Pascal_case#English):

> _e.g._ `IssueCard`

For interfaces, naming depends on context, but generally follows the same name in PascalCase:

> _e.g._ `categoryId` → `CategoryId`

Component props interfaces use the same name as the component with a `Props` suffix, to avoid naming collisions:

> _e.g._ `IssueCard` → `IssueCardProps`

## Goals

Every personal project I take on has something specific to practice — a framework, a styling approach, a concept. For this one, the focus was Clean Architecture: modeling entities, drawing clear boundaries between domain, infrastructure, and UI/app layers, and respecting the dependency rule so the domain stays isolated from everything above it. A Jira clone felt like a good fit since it has enough domain complexity to make that worthwhile.

Along the way, I also wanted to get more comfortable with Tailwind CSS and server-side rendering. I picked Remix over Next.js (which I already had experience with) because I liked its focus on web standards.

My most recent addition was a full design system, inspired by [Atlassian's Design System](https://atlassian.design/components/tokens/all-tokens#color-text) and its semantic color tokens. I built a base color palette, then layered semantic variables on top (`border`, `border-focus`, `border-selected`, `border-disabled`, etc.), each implemented independently per theme class (`theme-light`, `theme-dark`, `theme-dark-blue`, and so on). Switching themes is as simple as swapping the root element's class name — every themed variable updates automatically. So far I've built 6 fully customizable themes, and adding more is trivial. The idea: the hard part should be the creative decisions, not the technical plumbing.
