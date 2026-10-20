# School Management System (Demo)

A front-end demo of a school management system built with **Vue 3**, **Vite** and **Tailwind CSS**.

**Live demo:** https://luqmanarshad.github.io/demos/school/

## Features

- **Dashboard:** total students, today's attendance %, fees collected / pending, students per class.
- **Students:** search by name or roll number, filter by class, add and delete students.
- **Attendance:** mark each student as present, absent or on leave.
- **Fees:** list of pending fees with a one-click "Mark paid".

## Notes

- This is a demo with sample data. There is no backend or database; data is saved in the browser's `localStorage`, and "Reset demo data" restores the samples.
- Production versions of systems like this are built with Laravel (REST API, MySQL, RBAC) and Vue.js.

## Run locally

```bash
npm install
npm run dev
```

Build for production with `npm run build`.

## Author

Luqman Arshad, Full Stack Engineer (Laravel, Vue.js, Angular).
Portfolio: https://luqmanarshad.github.io/
