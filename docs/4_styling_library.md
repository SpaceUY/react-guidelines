---
title: Styling
layout: default
nav_order: 4
---

# Choosing the Right Styling Library for Your React Project

When it comes to styling your React project, we've standardized on **[ShadCN/UI](https://ui.shadcn.com/)** as our primary styling solution for consistency and maintainability across all projects.

## 🎯 **Our Standard: ShadCN/UI**

**ShadCN/UI** is our go-to choice for all React projects because it provides:

- **Consistent Design System**: Unified component library across all projects
- **Tailwind CSS Foundation**: Utility-first approach for maximum customization
- **Zero Abstraction**: Full control over every aspect of styling
- **Accessibility Built-in**: Components follow accessibility best practices
- **Easy Customization**: Modify colors, spacing, and components without limitations
- **Performance**: Lightweight and optimized for production

## 🚀 **Implementation Guidelines**

### **For All New Projects**
1. **Start with ShadCN/UI** as the base component library
2. **Use Tailwind CSS** for custom styling and layout
3. **Follow our design system** for colors, spacing, and typography
4. **Customize components** to match your project's visual identity

### **Component Customization**
```bash
# Install ShadCN/UI components
npx shadcn@latest add button
npx shadcn@latest add card
npx shadcn@latest add input
# ... add more as needed
```

### **Typography**
- Standardize font families and sizes
- Maintain proper hierarchy with consistent heading styles

## 💡 **Best Practices**

1. **Start Simple**: Begin with basic ShadCN/UI components
2. **Customize Gradually**: Modify components to match your design needs
3. **Maintain Consistency**: Use the same patterns across similar components
4. **Document Changes**: Keep track of customizations for team reference
5. **Performance First**: Avoid unnecessary CSS-in-JS for performance-critical applications

## 📚 **Resources**

- **[ShadCN/UI](https://ui.shadcn.com/)**: Official ShadCN/UI component library and documentation
- **[Animate UI](https://animate-ui.com/)**: Animation library for enhancing UI components with smooth transitions and effects
