# Digi-B1ND3R Web App

A modern digital design asset organizer that helps you store, manage, and access all your design resources in one place. Built with React, TypeScript, and Tailwind CSS.

## 🎨 Features

- **Color Management**: Store and organize color palettes, hex codes, and design tokens
- **Font Library**: Keep track of favorite fonts, pairings, and typography systems
- **Icon Collection**: Organize icon sets, SVGs, and graphic elements for easy access
- **Image Gallery**: Collect and manage reference images, inspiration, and assets
- **Modern UI**: Clean, responsive interface built with shadcn/ui components
- **Dark Theme**: Sleek dark mode design optimized for designers

## 🛠 Tech Stack

- **Frontend**: React 18.3.1 with TypeScript
- **Routing**: React Router v6
- **Styling**: Tailwind CSS with shadcn/ui components
- **Build Tool**: Vite 6.4.1
- **Icons**: Lucide React
- **Backend**: Supabase integration ready
- **Components**: Radix UI primitives

## 🚀 Quick Start

### Prerequisites

- Node.js 18+
- npm or yarn

### Installation

1. Clone the repository:

```bash
git clone <repository-url>
cd "Digi-B1ND3R Web App"
```

2. Install dependencies:

```bash
npm install
```

3. Start the development server:

```bash
npm run dev
```

4. Open your browser and navigate to `http://localhost:5173`

### Build for Production

```bash
npm run build
```

The build files will be generated in the `build/` directory.

## 📁 Project Structure

```
src/
├── components/
│   ├── ui/              # shadcn/ui components
│   └── figma/           # Figma-specific components
├── pages/               # Route components
│   ├── Home.tsx
│   ├── Colors.tsx
│   ├── Fonts.tsx
│   ├── Icons.tsx
│   ├── Images.tsx
│   └── Layout.tsx
├── utils/
│   └── supabase/        # Supabase utilities
├── routes.ts            # Route configuration
├── App.tsx              # Main app component
└── main.tsx             # App entry point
```

## 🎯 Usage

### Home Dashboard

The main dashboard provides an overview of all your design assets with quick access to each category.

### Color Management

- Store color palettes and hex codes
- Organize colors by projects or themes
- Export palettes in various formats

### Font Library

- Save font families and pairings
- Organize typography systems
- Track font sources and licensing

### Icon Collection

- Store SVG icons and icon sets
- Organize by category or project
- Quick search and filtering

### Image Gallery

- Collect reference images and inspiration
- Organize by mood boards or projects
- Tag and search functionality

## 🔧 Configuration

### Environment Variables

Create a `.env.local` file in the root directory:

```env
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```

### Supabase Setup

1. Create a new Supabase project
2. Configure the necessary tables for storing design assets
3. Update the environment variables with your Supabase credentials

## 🤝 Contributing

Please read our [Contributing Guide](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Attributions

- UI components from [shadcn/ui](https://ui.shadcn.com/) used under [MIT license](https://github.com/shadcn-ui/ui/blob/main/LICENSE.md)
- Design inspiration from [Figma](https://www.figma.com/design/r9hWEw3feKReyKTnbvijcn/Digi-B1ND3R-Web-App)
- Icons from [Lucide](https://lucide.dev/)

## 📧 Support

For support, please open an issue on GitHub or contact the development team.

---

**Digi-B1ND3R** - Organize your creative workflow, one asset at a time.
