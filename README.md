# Todo List App

A cross-platform Todo List application built with uni-app, supporting both WeChat Mini Program and H5 platforms.

## Features

- Add, edit, and delete tasks
- Mark tasks as completed
- Filter tasks (All, Active, Completed)
- Set due dates and priority levels
- Add notes to tasks
- Persistent storage across sessions
- Responsive design for different screen sizes

## Technology Stack

- **Framework**: uni-app
- **UI Framework**: Vue 3 (Composition API with script setup)
- **Language**: TypeScript
- **Build Tool**: Vite

## Project Structure

```
todo-list/
├── src/                    # Source code
│   ├── pages/              # Pages
│   │   ├── index/          # Main page (Todo list)
│   │   │   └── index.vue
│   │   └── edit/           # Edit page
│   │       └── edit.vue
│   ├── components/         # Components
│   │   ├── TodoItem.vue    # Single todo item component
│   │   └── TodoForm.vue    # Todo form component (add/edit)
│   ├── static/             # Static resources
│   ├── App.vue             # App entry component
│   ├── main.ts             # App entry file
│   ├── manifest.json       # App configuration
│   └── pages.json          # Page routing configuration
```

## Getting Started

### Prerequisites

- Node.js (>= 14.x)
- npm or yarn

### Installation

```bash
# Clone the repository
git clone https://github.com/jisuanjixue/todo-list.git

# Navigate to the project directory
cd todo-list

# Install dependencies
npm install
# or
yarn install
```

### Development

```bash
# Run for H5
npm run dev:h5
# or
yarn dev:h5

# Run for WeChat Mini Program
npm run dev:mp-weixin
# or
yarn dev:mp-weixin
```

### Build for Production

```bash
# Build for H5
npm run build:h5
# or
yarn build:h5

# Build for WeChat Mini Program
npm run build:mp-weixin
# or
yarn build:mp-weixin
```

## Screenshots

(Screenshots will be added here)

## License

MIT
