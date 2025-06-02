# Buenas Prácticas en React

## Tabla de Contenidos
1. [Principios Fundamentales](#principios-fundamentales)
2. [Estructura de Archivos](#estructura-de-archivos)
3. [Performance & Optimización](#performance--optimización)
4. [Legibilidad y Mantenibilidad](#legibilidad-y-mantenibilidad)
5. [Tipos y DTOs](#tipos-y-dtos)
6. [Gestión de Estado](#gestión-de-estado)
7. [Testing](#testing)
8. [Herramientas y Configuración](#herramientas-y-configuración)

---

## Principios Fundamentales

### DRY (Don't Repeat Yourself)
- **Evita duplicación de código**: Crea componentes reutilizables y hooks personalizados
- **Centraliza lógica común**: Usa custom hooks para lógica compartida
- **Constantes y configuración**: Mantén valores constantes en archivos separados

```tsx
// ❌ Malo - Repetición de lógica
const UserProfile = () => {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const fetchUser = async () => {
    setLoading(true);
    try {
      const response = await api.getUser();
      // lógica...
    } catch (err) {
      setError(err);
    } finally {
      setLoading(false);
    }
  };
};

// ✅ Bueno - Custom hook reutilizable
const useApiCall = (apiFunction) => {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [data, setData] = useState(null);
  
  const execute = async (...args) => {
    setLoading(true);
    setError(null);
    try {
      const result = await apiFunction(...args);
      setData(result);
      return result;
    } catch (err) {
      setError(err);
      throw err;
    } finally {
      setLoading(false);
    }
  };
  
  return { data, loading, error, execute };
};
```

### KISS (Keep It Simple, Stupid)
- **Componentes simples**: Un componente debe hacer una cosa y hacerla bien
- **Props claras**: Usa nombres descriptivos y evita props complejas
- **Evita over-engineering**: No agregues complejidad innecesaria

```tsx
// ❌ Malo - Componente complejo que hace muchas cosas
const UserDashboard = ({ userId, showAnalytics, enableNotifications }) => {
  // Lógica de usuario
  // Lógica de analytics
  // Lógica de notificaciones
  // Renderizado complejo
};

// ✅ Bueno - Componentes separados y simples
const UserProfile = ({ user }) => {
  return <div>{/* Solo mostrar perfil */}</div>;
};

const UserAnalytics = ({ analytics }) => {
  return <div>{/* Solo mostrar analytics */}</div>;
};

const UserDashboard = ({ userId }) => {
  const { user } = useUser(userId);
  const { analytics } = useAnalytics(userId);
  
  return (
    <div>
      <UserProfile user={user} />
      <UserAnalytics analytics={analytics} />
    </div>
  );
};
```

### Single Responsibility Principle
- **Una responsabilidad por componente**: Cada componente debe tener una razón para cambiar
- **Separación de concerns**: UI, lógica de negocio, y gestión de estado deben estar separados
- **Hooks especializados**: Cada hook debe manejar un aspecto específico

```tsx
// ❌ Malo - Muchas responsabilidades
const ProductCard = ({ productId }) => {
  const [product, setProduct] = useState(null);
  const [cart, setCart] = useState([]);
  const [user, setUser] = useState(null);
  const [analytics, setAnalytics] = useState({});
  
  // Fetch product, manage cart, track analytics, manage user...
};

// ✅ Bueno - Responsabilidades separadas
const useProduct = (productId) => {
  // Solo maneja productos
};

const useCart = () => {
  // Solo maneja carrito
};

const useAnalytics = () => {
  // Solo maneja analytics
};

const ProductCard = ({ productId }) => {
  const { product } = useProduct(productId);
  const { addToCart } = useCart();
  const { trackClick } = useAnalytics();
  
  return (
    <Card>
      {/* Solo renderizado */}
    </Card>
  );
};
```

---

## Estructura de Archivos

### Estructura Recomendada
```
src/
├── components/           # Componentes reutilizables
│   ├── ui/              # Componentes básicos de UI
│   │   ├── Button/
│   │   │   ├── index.ts
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   └── Button.module.css
│   │   └── Input/
│   └── common/          # Componentes comunes de negocio
├── pages/               # Páginas/Rutas principales
│   ├── Home/
│   ├── Products/
│   └── Profile/
├── features/            # Funcionalidades por dominio
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── types/
│   │   └── index.ts
│   ├── products/
│   └── user/
├── hooks/               # Custom hooks globales
├── services/            # API calls y servicios externos
├── types/               # Tipos TypeScript globales
├── utils/               # Funciones utilitarias
├── constants/           # Constantes de la aplicación
├── contexts/            # React Contexts
└── assets/              # Imágenes, iconos, etc.
```

### Convenciones de Nomenclatura
```tsx
// Componentes: PascalCase
const UserProfile = () => {};

// Hooks: camelCase con prefijo 'use'
const useUserData = () => {};

// Constantes: UPPER_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com';

// Archivos: kebab-case o PascalCase para componentes
user-profile.tsx
UserProfile.tsx

// Carpetas: kebab-case
user-management/
api-services/
```

---

## Performance & Optimización

### React.memo y Memoización
```tsx
// ✅ Memoizar componentes que reciben props complejas
const ProductCard = React.memo(({ product, onAddToCart }) => {
  return (
    <div>
      <h3>{product.name}</h3>
      <p>{product.price}</p>
      <Button onClick={() => onAddToCart(product.id)}>
        Agregar al Carrito
      </Button>
    </div>
  );
});

// ✅ useMemo para cálculos costosos
const ExpensiveComponent = ({ items }) => {
  const expensiveValue = useMemo(() => {
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]);
  
  return <div>{expensiveValue}</div>;
};

// ✅ useCallback para funciones que se pasan como props
const ParentComponent = () => {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback((id) => {
    // Lógica que no depende de count
    console.log('Clicked:', id);
  }, []); // Dependencias vacías si no cambia
  
  return <ChildComponent onClick={handleClick} />;
};
```

### Lazy Loading y Code Splitting
```tsx
// ✅ Lazy loading de componentes
const LazyProductPage = lazy(() => import('./pages/ProductPage'));
const LazyUserProfile = lazy(() => import('./components/UserProfile'));

const App = () => {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/products" element={<LazyProductPage />} />
        <Route path="/profile" element={<LazyUserProfile />} />
      </Routes>
    </Suspense>
  );
};

// ✅ Preload de componentes importantes
const preloadProductPage = () => {
  import('./pages/ProductPage');
};

// Preload cuando el usuario hace hover
<Link 
  to="/products" 
  onMouseEnter={preloadProductPage}
>
  Productos
</Link>
```

### Optimización de Re-renders
```tsx
// ❌ Malo - Crea nuevo objeto en cada render
const BadComponent = () => {
  const [user, setUser] = useState(null);
  
  return (
    <UserProfile 
      user={user}
      config={{ theme: 'dark', size: 'large' }} // ❌ Nuevo objeto cada vez
    />
  );
};

// ✅ Bueno - Objeto estable
const config = { theme: 'dark', size: 'large' }; // Fuera del componente

const GoodComponent = () => {
  const [user, setUser] = useState(null);
  
  const config = useMemo(() => ({ 
    theme: 'dark', 
    size: 'large' 
  }), []); // O usar constante externa
  
  return <UserProfile user={user} config={config} />;
};
```

---

## Legibilidad y Mantenibilidad

### Nombres Descriptivos
```tsx
// ❌ Malo
const u = useUser();
const handleClick = () => {};
const data = fetchData();

// ✅ Bueno
const currentUser = useUser();
const handleAddToCartClick = () => {};
const productListData = fetchProductList();
```

### Composición vs Herencia
```tsx
// ✅ Composición - Más flexible
const Card = ({ children, className = '' }) => (
  <div className={`card ${className}`}>
    {children}
  </div>
);

const ProductCard = ({ product }) => (
  <Card className="product-card">
    <img src={product.image} alt={product.name} />
    <h3>{product.name}</h3>
    <p>{product.price}</p>
  </Card>
);

const UserCard = ({ user }) => (
  <Card className="user-card">
    <Avatar src={user.avatar} />
    <h3>{user.name}</h3>
    <p>{user.email}</p>
  </Card>
);
```

### Props Drilling - Soluciones
```tsx
// ❌ Malo - Props drilling
const App = () => {
  const [user, setUser] = useState(null);
  return <HomePage user={user} setUser={setUser} />;
};

const HomePage = ({ user, setUser }) => {
  return <UserSection user={user} setUser={setUser} />;
};

const UserSection = ({ user, setUser }) => {
  return <UserProfile user={user} setUser={setUser} />;
};

// ✅ Bueno - Context para estado global
const UserContext = createContext();

const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
};

const useUser = () => {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser debe usarse dentro de UserProvider');
  }
  return context;
};

// ✅ Bueno - Props específicas y composición
const UserProfile = () => {
  const { user } = useUser();
  return <div>{user?.name}</div>;
};
```

---

## Tipos y DTOs

### TypeScript Interfaces y Types
```tsx
// ✅ Interfaces para objetos que pueden extenderse
interface User {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
}

interface AdminUser extends User {
  permissions: Permission[];
  role: 'admin' | 'super-admin';
}

// ✅ Types para uniones y tipos más complejos
type Status = 'loading' | 'success' | 'error';
type Theme = 'light' | 'dark' | 'auto';

// ✅ DTOs para datos de API
interface CreateUserDTO {
  name: string;
  email: string;
  password: string;
}

interface UpdateUserDTO {
  name?: string;
  email?: string;
}

interface UserResponseDTO {
  id: string;
  name: string;
  email: string;
  created_at: string; // Como viene de la API
}

// ✅ Transformers para convertir DTOs a modelos
const transformUserResponse = (dto: UserResponseDTO): User => ({
  id: dto.id,
  name: dto.name,
  email: dto.email,
  createdAt: new Date(dto.created_at),
});
```

### Props Typing
```tsx
// ✅ Props bien tipadas
interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
  className?: string;
}

const Button: React.FC<ButtonProps> = ({
  children,
  variant = 'primary',
  size = 'md',
  disabled = false,
  onClick,
  className = '',
}) => {
  return (
    <button
      className={`btn btn-${variant} btn-${size} ${className}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
};

// ✅ Props con discriminated unions
interface LoadingState {
  status: 'loading';
}

interface SuccessState {
  status: 'success';
  data: User[];
}

interface ErrorState {
  status: 'error';
  error: string;
}

type DataState = LoadingState | SuccessState | ErrorState;

const DataDisplay: React.FC<{ state: DataState }> = ({ state }) => {
  switch (state.status) {
    case 'loading':
      return <Spinner />;
    case 'success':
      return <UserList users={state.data} />;
    case 'error':
      return <ErrorMessage message={state.error} />;
  }
};
```

### Generic Types
```tsx
// ✅ Hooks genéricos reutilizables
interface ApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}

const useApi = <T>() => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const fetchData = async (url: string): Promise<T> => {
    setLoading(true);
    try {
      const response = await fetch(url);
      const result: ApiResponse<T> = await response.json();
      setData(result.data);
      return result.data;
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Error desconocido');
      throw err;
    } finally {
      setLoading(false);
    }
  };
  
  return { data, loading, error, fetchData };
};

// Uso específico
const UserList = () => {
  const { data: users, loading, fetchData } = useApi<User[]>();
  
  useEffect(() => {
    fetchData('/api/users');
  }, []);
  
  // ...
};
```

---

## Gestión de Estado

### Estado Local vs Global
```tsx
// ✅ Estado local para UI
const Modal = () => {
  const [isOpen, setIsOpen] = useState(false); // UI state - local
  
  return (
    <>
      <Button onClick={() => setIsOpen(true)}>Abrir Modal</Button>
      {isOpen && <ModalContent onClose={() => setIsOpen(false)} />}
    </>
  );
};

// ✅ Estado global para datos compartidos
const UserProvider = ({ children }) => {
  const [currentUser, setCurrentUser] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  
  // Este estado se comparte en toda la app
  return (
    <UserContext.Provider value={{ currentUser, isAuthenticated }}>
      {children}
    </UserContext.Provider>
  );
};
```

### Reducers para Lógica Compleja
```tsx
// ✅ useReducer para estado complejo
interface CartState {
  items: CartItem[];
  total: number;
  isLoading: boolean;
}

type CartAction =
  | { type: 'ADD_ITEM'; payload: CartItem }
  | { type: 'REMOVE_ITEM'; payload: string }
  | { type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }
  | { type: 'CLEAR_CART' }
  | { type: 'SET_LOADING'; payload: boolean };

const cartReducer = (state: CartState, action: CartAction): CartState => {
  switch (action.type) {
    case 'ADD_ITEM':
      const existingItem = state.items.find(item => item.id === action.payload.id);
      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + action.payload.quantity }
              : item
          ),
        };
      }
      return {
        ...state,
        items: [...state.items, action.payload],
      };
    
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload),
      };
    
    default:
      return state;
  }
};

const useCart = () => {
  const [state, dispatch] = useReducer(cartReducer, {
    items: [],
    total: 0,
    isLoading: false,
  });
  
  const addItem = (item: CartItem) => {
    dispatch({ type: 'ADD_ITEM', payload: item });
  };
  
  return { ...state, addItem };
};
```

---

## Testing

### Unit Tests para Componentes
```tsx
// ProductCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { ProductCard } from './ProductCard';

const mockProduct = {
  id: '1',
  name: 'Producto Test',
  price: 100,
  image: 'test-image.jpg',
};

describe('ProductCard', () => {
  it('debe renderizar la información del producto', () => {
    render(<ProductCard product={mockProduct} />);
    
    expect(screen.getByText('Producto Test')).toBeInTheDocument();
    expect(screen.getByText('$100')).toBeInTheDocument();
  });
  
  it('debe llamar onAddToCart cuando se hace click', () => {
    const mockAddToCart = jest.fn();
    render(
      <ProductCard 
        product={mockProduct} 
        onAddToCart={mockAddToCart} 
      />
    );
    
    fireEvent.click(screen.getByText('Agregar al Carrito'));
    expect(mockAddToCart).toHaveBeenCalledWith('1');
  });
});
```

### Tests para Custom Hooks
```tsx
// useCart.test.ts
import { renderHook, act } from '@testing-library/react';
import { useCart } from './useCart';

describe('useCart', () => {
  it('debe agregar item al carrito', () => {
    const { result } = renderHook(() => useCart());
    
    act(() => {
      result.current.addItem({
        id: '1',
        name: 'Producto',
        price: 100,
        quantity: 1,
      });
    });
    
    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].name).toBe('Producto');
  });
});
```

---

## Herramientas y Configuración

### ESLint Configuration
```json
// .eslintrc.js
{
  "extends": [
    "react-app",
    "react-app/jest",
    "@typescript-eslint/recommended",
    "prettier"
  ],
  "rules": {
    "react/jsx-boolean-value": ["error", "never"],
    "react/jsx-curly-brace-presence": ["error", "never"],
    "react/no-array-index-key": "error",
    "react-hooks/exhaustive-deps": "error",
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "off",
    "prefer-const": "error",
    "no-console": "warn"
  }
}
```

### Prettier Configuration
```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

### Husky y Git Hooks
```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "pre-push": "npm test"
    }
  },
  "lint-staged": {
    "src/**/*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "git add"
    ]
  }
}
```
