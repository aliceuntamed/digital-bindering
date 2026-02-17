# Agent Instructions for Digi-B1ND3R Web App

This document provides guidance for AI assistants working on the Digi-B1ND3R Web App project.

## Interaction Guidelines

When interacting with user follow these guidelines:

- Keep in mind (memory) the user is on the free plan, so offer responses and solutions accordingly.
- Provide clear explanations. Be patient, user is a beginner and this is their first app.
- Use proper markdown formatting
- Maintain a friendly and supportive tone
- Be opininated, without needing to be asked directly. If there is a mistake or improvement that can be made, suggest it openly. Also often suggest follow ups, user appreciates as much guidance as possible.
- Always use memory. Recall previous conversations and context.

## Project Overview

Digi-B1ND3R is a modern, highly detailed, and feature-rich digital design asset organizer built with React, TypeScript, and Tailwind CSS. It helps designers store, manage, and access their design resources (colors, fonts, icons, images) in one centralized location.

## Tech Stack & Architecture

### Core Technologies

- **React 18.3.1** with TypeScript for component-based architecture
- **React Router v6** for client-side routing
- **Vite 6.4.1** as the build tool and dev server
- **Tailwind CSS** for styling with shadcn/ui components
- **Supabase** for backend services and database

### Key Libraries

- **Radix UI**: Primitive components for accessible UI
- **Lucide React**: Icon library
- **React Hook Form**: Form management
- **Recharts**: Data visualization
- **Sonner**: Toast notifications

## Project Structure

```
src/
├── components/
│   ├── ui/              # shadcn/ui reusable components
│   └── figma/           # Figma-specific components
├── pages/               # Route-level components
│   ├── Layout.tsx       # Main app layout with navigation
│   ├── Home.tsx         # Dashboard overview
│   ├── Colors.tsx       # Color management page
│   ├── Fonts.tsx        # Font library page
│   ├── Icons.tsx        # Icon collection page
│   └── Images.tsx       # Image gallery page
├── utils/
│   └── supabase/        # Supabase client and utilities
├── routes.ts            # Route configuration
├── App.tsx              # Root component with router
└── main.tsx             # Application entry point
```

## Design System

### Color Scheme

- **Primary Dark**: `#041F1E` (background)
- **Secondary Dark**: `#424651` (header)
- **Text Primary**: White (`#FFFFFF`)
- **Text Secondary**: Gray (`#9CA3AF`)
- **Accent Colors**: `#DBD3AD`, `#67597A`, `#FF6B6B`, `#6E8894
  , along with various shades of these colors for different states and effects

### Typography

- **Font Family**: Mono font for branding (`font-mono`)
- **Headings**: Large, bold text with high contrast
- **Body Text**: System fonts with proper contrast ratios

### Component Patterns

- **Cards**: White background with rounded corners and borders
- **Navigation**: Dark theme with active state highlighting
- **Buttons**: Consistent styling with hover states
- **Forms**: shadcn/ui components with proper validation

## Development Guidelines

### Code Style

- Use TypeScript for all new components
- Follow React functional component patterns
- Use Tailwind classes for styling (avoid inline styles)
- Implement proper accessibility with ARIA labels
- Use semantic HTML elements

### Component Development

- Create reusable components in `components/ui/`
- Use proper TypeScript interfaces for props
- Implement loading states and error handling
- Follow the existing naming conventions
- Use Lucide React icons consistently

### State Management

- Use React hooks for local state
- Implement proper data fetching patterns
- Consider React Query for server state if needed
- Use Supabase for persistent data storage

## Routing & Navigation

### Current Routes

- `/` - Home dashboard
- `/colors` - Color management
- `/fonts` - Font library
- `/icons` - Icon collection
- `/images` - Image gallery

### Navigation Pattern

- Use `NavLink` for active state styling
- Maintain consistent navigation in Layout component
- Implement proper breadcrumb navigation if needed

## Database Schema (Supabase)

### Expected Tables

- `colors` - Store color palettes and hex codes
- `fonts` - Font families and typography data
- `icons` - Icon sets and SVG data
- `images` - Image references and metadata
- `projects` - Project organization
- `users` - User management

## Common Tasks

### Adding a New Page

1. Create component in `src/pages/`
2. Add route in `src/routes.ts`
3. Update navigation in `src/pages/Layout.tsx`
4. Add any necessary database tables

### Creating UI Components

1. Use shadcn/ui patterns in `src/components/ui/`
2. Follow Radix UI best practices
3. Implement proper TypeScript types
4. Add responsive design with Tailwind

### Database Integration

1. Set up Supabase client in `src/utils/supabase/`
2. Create proper TypeScript interfaces
3. Implement error handling
4. Use React Query patterns for data fetching

## Testing & Quality

### Testing Strategy

- Unit tests for utility functions
- Component tests for UI components
- Integration tests for user flows
- Accessibility testing with screen readers

### Code Quality

- Use ESLint and Prettier configurations
- Implement proper TypeScript strict mode
- Follow React best practices
- Ensure accessibility compliance

## Performance Considerations

### Optimization

- Use React.memo for expensive components
- Implement proper loading states
- Optimize images and assets
- Use code splitting for large components

### Bundle Size

- Monitor bundle size with Vite analytics
- Use dynamic imports for heavy dependencies
- Optimize Supabase queries
- Implement proper caching strategies

## Deployment

### Build Process

- Use `npm run build` for production
- Ensure environment variables are properly set
- Test build locally before deployment
- Verify Supabase connection in production

### Environment Setup

- Configure Supabase project
- Set up environment variables
- Configure hosting platform
- Set up custom domain if needed

## Troubleshooting

### Common Issues

- **Supabase Connection**: Check environment variables and network policies
- **Routing**: Verify route configuration and NavLink usage
- **Styling**: Ensure Tailwind classes are properly applied
- **TypeScript**: Check for proper type definitions

### Debugging Tips

- Use React DevTools for component inspection
- Check browser console for errors
- Verify network requests in DevTools
- Use Supabase dashboard for database issues

## Future Enhancements

### Planned Features

- Real-time collaboration
- Advanced search and filtering
- Export/import functionality
- Integration with design tools
- Mobile app development
- Additional asset types (videos, documents, audio, cms, inventory, etc.)

### Technical Improvements

- State management with Zustand/Redux
- Advanced caching strategies
- Performance monitoring
- Automated testing pipeline

---

When working on this project, always prioritize user experience, accessibility, and code maintainability. Follow the established patterns and contribute to the documentation as you add new features.
