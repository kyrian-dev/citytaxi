# CityTaxi 🚕

A responsive taxi booking portfolio demo built with Vue 3, JavaScript, Vite, and CSS.

## Features

- Mobile-friendly navy-and-gold design
- English and German language switching
- Pickup and destination fields
- Date, time, and passenger selection
- Validation to prevent bookings in the past
- Illustrative fare estimate based on manually entered distance
- Demo booking confirmation and trip summary
- Dashboard storing the last 10 demo bookings in localStorage
- Delete saved demo bookings
- Custom taxi logo and favicon
- Placeholder WhatsApp and phone contact options

## Demo limitations

This project demonstrates a booking interface. It does not send booking
requests or reserve taxis.

Fare estimates use an illustrative formula:
€4 base fare + €2 per kilometre. They are not real quotes.

Bookings are stored only in the current browser, with no account or server
storage. Use fictional details when testing.

WhatsApp and phone buttons are disabled until real operator contact
details are configured.

Live maps, route distances, real fares, dispatch, and payments require
external integrations and a back end.

## Built with

- Vue 3 — Composition API
- JavaScript
- Vite
- CSS
- Browser localStorage

## Run locally

Install Node.js and npm, then open the project folder in your terminal.

Install dependencies:

    npm install

Start the development server:

    npm run dev

Open the local address shown in the terminal.

### Windows PowerShell

If PowerShell blocks npm scripts, use:

    npm.cmd install
    npm.cmd run dev

## Production build

Build the project:

    npm run build

Preview the production build:

    npm run preview

The production files are generated in the `dist` folder.

In Windows PowerShell, use `npm.cmd` instead of `npm` if needed.

## Project structure

    public/
      images/
        taxi.jpg
      logo.svg
    src/
      App.vue
      main.js
      style.css
    index.html
    package.json

## Image credits

Before publishing, add the taxi photo's photographer, source link,
and any attribution required by its license.