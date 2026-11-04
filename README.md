# Agenda de Contatos — Cypress E2E

Suite de testes end-to-end da Agenda de Contatos.

[![Cypress E2E Tests](https://github.com/viniciussilva2504/teste_Cypress_AgendaDeContatos/actions/workflows/cypress.yml/badge.svg)](https://github.com/viniciussilva2504/teste_Cypress_AgendaDeContatos/actions/workflows/cypress.yml)

This repository is the Cypress suite for the deployed **Agenda de Contatos** app. It checks creating, editing, and deleting contacts, including the JSON sent to the API, plus the same add flow on a phone-sized viewport.

![Agenda de Contatos](docs/app-screenshot.png)

## App under test

| | |
|---|---|
| App | [ebac-agenda-contatos-tan.vercel.app](https://ebac-agenda-contatos-tan.vercel.app/) |
| `baseUrl` | `https://ebac-agenda-contatos-tan.vercel.app` (`cypress.config.js`) |
| API | `https://api-ebac.vercel.app/api/contatos` |

The suite treats that API as a shared instance. Every spec stubs it with `cy.intercept()` and JSON fixtures, and asserts against those stubs.

The screenshot above is the deployed UI with whatever contacts are on the shared API. Specs assert fixture names (`Vinicius Silva`, `Maria Souza`, and the edited values below).

## Scope

`cypress/e2e/agenda-contatos/home.cy.js` defines **10 tests** in four contexts:

- **Adicionar contato** — one valid contact, two contacts, empty required fields, invalid email
- **Editar contato** — change the name, change the email
- **Excluir contato** — remove the only contact, then assert the list is empty
- **Responsividade (mobile)** — form at 375×667, and adding a contact at that size

Selectors follow the deployed markup: `input[type="text"]`, `input[type="email"]`, `input[type="tel"]`, and the classes `.adicionar`, `.edit`, `.delete`, and `.alterar`.

## Test approach

Positive cases fill the form and assert the contact list. Negative cases submit empty required fields, or the invalid email from `invalid-contact.json` (`email-invalido`), and assert that no edit control appears.

Each context registers a GET intercept, then calls `cy.visit('/')` and `cy.wait('@getContacts')`. Create, edit, and delete register their own intercepts and wait on those aliases. Waits in the spec use those aliases.

POST and PUT send `{ contato: { name, email, phone } }`. The spec reads `request.body` (string or object) and asserts the fields that the test changed. Edit and delete responses come from fixtures, and the UI assertions follow that fixture data.

`cypress/support/commands.js` defines `addContact`, `deleteFirstContact`, and `editFirstContact`. The spec types into the form in each test.

`cypress/fixtures/example.json` is the Cypress sample fixture. The tests load the fixtures in the table below.

## Coverage

IDs follow the order of the `it` blocks in `cypress/e2e/agenda-contatos/home.cy.js`.

| ID | What it checks | Spec file |
|---|---|---|
| AC-01 | Valid contact: POST `contato` includes name and email; name and email are shown; one `.edit` control | `cypress/e2e/agenda-contatos/home.cy.js` |
| AC-02 | Two contacts: each POST `contato` includes that contact's name; both names are shown; two `.edit` controls | `cypress/e2e/agenda-contatos/home.cy.js` |
| AC-03 | Empty required fields: clicking Adicionar leaves the list without an `.edit` control | `cypress/e2e/agenda-contatos/home.cy.js` |
| AC-04 | Invalid email (`email-invalido`): clicking Adicionar leaves the list without an `.edit` control | `cypress/e2e/agenda-contatos/home.cy.js` |
| AC-05 | Edit name: PUT `contato.name` is `Vinicius Editado`; the new name is shown and `Vinicius Silva` is gone | `cypress/e2e/agenda-contatos/home.cy.js` |
| AC-06 | Edit email: PUT `contato.email` is `editado@teste.com`; that email is shown | `cypress/e2e/agenda-contatos/home.cy.js` |
| AC-07 | Delete: the DELETE alias is waited; the fixture contact name is removed | `cypress/e2e/agenda-contatos/home.cy.js` |
| AC-08 | Delete the only contact: no `.edit` or `.delete` control remains | `cypress/e2e/agenda-contatos/home.cy.js` |
| AC-09 | At 375×667 the name, email, phone, and Adicionar controls are visible | `cypress/e2e/agenda-contatos/home.cy.js` |
| AC-10 | At 375×667 a valid contact can be added and its name is shown | `cypress/e2e/agenda-contatos/home.cy.js` |

Fixtures the spec loads:

| Fixture | Role |
|---|---|
| `contact.json` | Valid name, email, and phone typed into the form |
| `invalid-contact.json` | Invalid email used by AC-04 |
| `api-contacts-empty.json` | GET response: empty list |
| `api-contacts-one.json` | GET response: one contact |
| `api-contact-created.json` | POST response: one contact |
| `api-contact-created-2.json` | POST response: two contacts |
| `api-contact-edited-name.json` | PUT response: edited name |
| `api-contact-edited-email.json` | PUT response: edited email |

## Run locally

CI uses Node.js 18. Cypress 12.6.0 is the locked version.

```bash
npm ci
```

| Script | Command | What it does |
|---|---|---|
| `npm run cy:open` | `cypress open` | Interactive runner |
| `npm run cy:run` | `cypress run` | Headless run in the Electron browser bundled with Cypress |
| `npm run cy:run:headed` | `cypress run --headed` | Headed run |
| `npm run cy:report` | `cypress run --reporter mochawesome` | Headless run, reporter set on the command as well as in config |

`npm run cy:run` already uses the Mochawesome reporter from `cypress.config.js`. `npm run cy:report` passes the same reporter on the CLI.

## Reports

`cypress.config.js` writes a Mochawesome HTML file and a JSON file to `cypress/reports/`, named `mochawesome_<ddmmyyyy_HHMMss>.html` and `.json`.

Failed tests save screenshots (`screenshotOnRunFailure: true`) under `cypress/screenshots/`. Video is on (`video: true`) and is written to `cypress/videos/`.

Those three folders are listed in `.gitignore`. A saved example of the HTML report is in `docs/test-report-full.png`.

![Sample Mochawesome report](docs/test-report-full.png)

## CI

[`.github/workflows/cypress.yml`](.github/workflows/cypress.yml) runs on:

- `push` to `exercicio-cypress` and `main`
- `pull_request` targeting `exercicio-cypress` and `main`
- `workflow_dispatch`

The job runs on `ubuntu-24.04` with Node.js 18. It installs from `package-lock.json` (`npm ci`) and runs the suite headless in Chrome against the deployed `baseUrl`.

Artifacts, kept for 7 days:

- `cypress-report` — `cypress/reports` (uploaded on every run)
- `cypress-screenshots` — `cypress/screenshots` (uploaded when the job fails)

## Tech stack

Versions below are the ones resolved in `package-lock.json`, except Node.js and the runner, which come from the workflow.

| Tool | Version |
|---|---|
| Cypress | 12.6.0 |
| Mochawesome | 7.1.3 |
| Node.js (CI) | 18 |
| Browser (CI) | Chrome, headless |
| Browser (`npm run cy:run`) | Electron bundled with Cypress 12.6.0 |
| Runner | GitHub Actions, `ubuntu-24.04` |

## Project layout

```
cypress/
├── e2e/agenda-contatos/home.cy.js
├── fixtures/
├── support/commands.js
├── support/e2e.js
├── reports/        # Mochawesome output (gitignored)
├── screenshots/    # failure screenshots (gitignored)
└── videos/         # run videos (gitignored)
cypress.config.js
.github/workflows/cypress.yml
docs/               # README images
```

## Author

**Vinicius Jesus da Silva**
[linkedin.com/in/vjsilva2504](https://linkedin.com/in/vjsilva2504) · [portfolio-ebon-nine-95.vercel.app](https://portfolio-ebon-nine-95.vercel.app)
