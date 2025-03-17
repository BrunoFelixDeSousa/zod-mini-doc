# Documentação Zod: Do Básico ao Avançado

## Sumário
1. [Introdução ao Zod](#introdução-ao-zod)
2. [Instalação e Configuração](#instalação-e-configuração)
3. [Tipos Básicos](#tipos-básicos)
4. [Validações Personalizadas](#validações-personalizadas)
5. [Objetos e Schemas Complexos](#objetos-e-schemas-complexos)
6. [Arrays e Tuplas](#arrays-e-tuplas)
7. [Unions e Intersections](#unions-e-intersections)
8. [Inferência de Tipos](#inferência-de-tipos)
9. [Transformações](#transformações)
10. [Validações Assíncronas](#validações-assíncronas)
11. [Integrações com React e Form Handling](#integrações-com-react-e-form-handling)
12. [Casos de Uso Avançados](#casos-de-uso-avançados)

## Introdução ao Zod

O Zod é uma biblioteca de validação de schema com tipagem forte para TypeScript. Ele permite definir esquemas de validação de dados com uma API fluente e proporciona segurança de tipos em tempo de compilação.

### Principais características:

- **Zero dependências**
- **Tipagem forte**: Zod infere automaticamente tipos TypeScript
- **API declarativa e encadeável**
- **Mensagens de erro personalizáveis**
- **Ambiente agnostico**: Funciona no navegador e no Node.js

## Instalação e Configuração

```bash
# Usando npm
npm install zod

# Usando yarn
yarn add zod

# Usando pnpm
pnpm add zod
```

Configuração do TypeScript (tsconfig.json):

```json
{
  "compilerOptions": {
    "strict": true,
    "strictNullChecks": true
  }
}
```

Importação básica:

```typescript
import { z } from 'zod';
```

## Tipos Básicos

### String

```typescript
// Definição básica
const stringSchema = z.string();
stringSchema.parse("Hello"); // Retorna "Hello"
stringSchema.parse(123); // Lança erro ZodError

// Com validações
const emailSchema = z.string().email();
const urlSchema = z.string().url();
const minSchema = z.string().min(5);
const maxSchema = z.string().max(10);
const lengthSchema = z.string().length(8);
const regexSchema = z.string().regex(/^\d{2}-\d{4}-\d{4}$/);
const nonemptySchema = z.string().nonempty(); // Garante string não vazia

// Combinando validações
const usernameSchema = z.string()
  .min(3, "Username deve ter pelo menos 3 caracteres")
  .max(20, "Username deve ter no máximo 20 caracteres")
  .regex(/^[a-zA-Z0-9_]+$/, "Username deve conter apenas letras, números e underscore");
```

### Number

```typescript
// Definição básica
const numberSchema = z.number();
numberSchema.parse(123); // Retorna 123
numberSchema.parse("123"); // Lança erro

// Com validações
const positiveSchema = z.number().positive();
const negativeSchema = z.number().negative();
const intSchema = z.number().int();
const minSchema = z.number().min(5);
const maxSchema = z.number().max(10);
const gtSchema = z.number().gt(0);  // maior que
const gteSchema = z.number().gte(0); // maior ou igual a
const ltSchema = z.number().lt(100); // menor que
const lteSchema = z.number().lte(100); // menor ou igual a

// Validação com múltiplas regras
const scoreSchema = z.number()
  .int()
  .gte(0, "Pontuação deve ser positiva")
  .lte(100, "Pontuação máxima é 100");
```

### Boolean

```typescript
const boolSchema = z.boolean();
boolSchema.parse(true); // Retorna true
boolSchema.parse("true"); // Lança erro
```

### Date

```typescript
const dateSchema = z.date();
dateSchema.parse(new Date()); // Retorna objeto Date
dateSchema.parse("2023-01-01"); // Lança erro, precisa ser um objeto Date

// Com validações
const minDateSchema = z.date().min(new Date("2020-01-01"));
const maxDateSchema = z.date().max(new Date("2025-12-31"));

// Validação de data dentro de um intervalo
const dateRangeSchema = z.date()
  .min(new Date("2023-01-01"), "Data deve ser após 01/01/2023")
  .max(new Date("2023-12-31"), "Data deve ser antes de 31/12/2023");
```

### Null e Undefined

```typescript
const nullSchema = z.null();
const undefinedSchema = z.undefined();

// Valores específicos
const literalSchema = z.literal("hello");
literalSchema.parse("hello"); // Retorna "hello"
literalSchema.parse("world"); // Lança erro
```

### Opcionais e Nullable

```typescript
// Opcional (string | undefined)
const optionalString = z.string().optional();
optionalString.parse(undefined); // Retorna undefined
optionalString.parse("hello"); // Retorna "hello"

// Nullable (string | null)
const nullableString = z.string().nullable();
nullableString.parse(null); // Retorna null
nullableString.parse("hello"); // Retorna "hello"

// Valor padrão
const defaultString = z.string().default("padrão");
defaultString.parse(undefined); // Retorna "padrão"
```

## Validações Personalizadas

```typescript
// Validação personalizada com .refine()
const evenNumberSchema = z.number()
  .refine(n => n % 2 === 0, {
    message: "O número deve ser par"
  });

// Múltiplas validações encadeadas
const passwordSchema = z.string()
  .min(8, "Senha deve ter pelo menos 8 caracteres")
  .refine(
    password => /[A-Z]/.test(password),
    "Senha deve conter pelo menos uma letra maiúscula"
  )
  .refine(
    password => /[a-z]/.test(password),
    "Senha deve conter pelo menos uma letra minúscula"
  )
  .refine(
    password => /[0-9]/.test(password),
    "Senha deve conter pelo menos um número"
  )
  .refine(
    password => /[^A-Za-z0-9]/.test(password),
    "Senha deve conter pelo menos um caractere especial"
  );

// Exemplo de uso:
try {
  passwordSchema.parse("abc123");
} catch (error) {
  console.log(error.errors);
  // [
  //   "Senha deve ter pelo menos 8 caracteres",
  //   "Senha deve conter pelo menos uma letra maiúscula",
  //   "Senha deve conter pelo menos um caractere especial"
  // ]
}
```

### Validação com Super Refine (acesso ao contexto)

```typescript
const formSchema = z.object({
  password: z.string(),
  confirmPassword: z.string()
}).superRefine(({ password, confirmPassword }, ctx) => {
  if (password !== confirmPassword) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: "As senhas não coincidem",
      path: ["confirmPassword"]
    });
  }
});

// Exemplo de uso:
try {
  formSchema.parse({
    password: "Secret123!",
    confirmPassword: "Secret456!"
  });
} catch (error) {
  console.log(error.errors);
  // ["As senhas não coincidem"]
}
```

## Objetos e Schemas Complexos

### Objetos Básicos

```typescript
const userSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
  role: z.enum(["admin", "user", "guest"])
});

// Exemplo de uso:
const validUser = {
  id: 1,
  name: "John Doe",
  email: "john@example.com",
  role: "admin"
};

userSchema.parse(validUser);
```

### Campos Opcionais e Parciais

```typescript
// Definindo campos específicos como opcionais
const productSchema = z.object({
  id: z.number(),
  name: z.string(),
  price: z.number().positive(),
  description: z.string().optional(),
  tags: z.array(z.string()).optional()
});

// Tornando todos os campos opcionais
const partialProductSchema = productSchema.partial();
// Equivalente a:
// z.object({
//   id: z.number().optional(),
//   name: z.string().optional(),
//   price: z.number().positive().optional(),
//   description: z.string().optional().optional(), // ainda é só opcional
//   tags: z.array(z.string()).optional().optional() // ainda é só opcional
// });

// Tornando campos específicos opcionais
const updateProductSchema = productSchema.partial({
  name: true,
  price: true,
  description: true
});
```

### Campos Obrigatórios

```typescript
// Tornando todos os campos obrigatórios
const requiredProductSchema = partialProductSchema.required();

// Tornando campos específicos obrigatórios
const createProductSchema = partialProductSchema.required({
  name: true,
  price: true
});
```

### Estendendo Schemas

```typescript
const basePersonSchema = z.object({
  name: z.string(),
  age: z.number()
});

const employeeSchema = basePersonSchema.extend({
  department: z.string(),
  salary: z.number().positive()
});

// Exemplo de uso:
employeeSchema.parse({
  name: "João",
  age: 30,
  department: "Engenharia",
  salary: 5000
});
```

### Campos Aninhados

```typescript
const addressSchema = z.object({
  street: z.string(),
  city: z.string(),
  state: z.string().length(2),
  zipCode: z.string().regex(/^\d{5}-\d{3}$/)
});

const customerSchema = z.object({
  name: z.string(),
  email: z.string().email(),
  address: addressSchema
});

// Exemplo de uso:
customerSchema.parse({
  name: "Maria Silva",
  email: "maria@example.com",
  address: {
    street: "Rua das Flores, 123",
    city: "São Paulo",
    state: "SP",
    zipCode: "01234-567"
  }
});
```

### Catch e Safeguard

```typescript
// .parse() lança erro se inválido
try {
  userSchema.parse({ /* dados inválidos */ });
} catch (error) {
  console.log(error.errors);
}

// .safeParse() retorna um objeto com sucesso ou erro
const result = userSchema.safeParse({ /* dados */ });
if (result.success) {
  // Dados validados disponíveis em result.data
  console.log(result.data);
} else {
  // Erros disponíveis em result.error
  console.log(result.error.errors);
}
```

## Arrays e Tuplas

### Arrays

```typescript
// Array de strings
const stringArraySchema = z.array(z.string());
stringArraySchema.parse(["a", "b", "c"]); // OK
stringArraySchema.parse(["a", 1, "c"]); // Erro

// Array com validações
const nonEmptyArray = z.array(z.string()).nonempty();
const minMaxArray = z.array(z.number()).min(1).max(10);

// Array de objetos
const userArraySchema = z.array(
  z.object({
    id: z.number(),
    name: z.string()
  })
);

// Exemplo de uso:
userArraySchema.parse([
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" }
]);
```

### Tuplas

```typescript
// Tupla com tipos específicos por posição
const tupleSchema = z.tuple([
  z.string(),
  z.number(),
  z.boolean()
]);

tupleSchema.parse(["hello", 42, true]); // OK
tupleSchema.parse(["hello", 42]); // Erro: esperado 3 elementos
tupleSchema.parse(["hello", "42", true]); // Erro: segundo elemento deve ser number

// Tupla com elementos opcionais adicionais (rest)
const pointWithMetadataSchema = z.tuple([z.number(), z.number()])
  .rest(z.string()); // pode ter mais strings depois dos dois números

// Válido: [10, 20]
// Válido: [10, 20, "label", "color"]
// Inválido: [10, 20, true]

pointWithMetadataSchema.parse([10, 20, "red", "large"]); // OK
```

## Unions e Intersections

### Union Types

```typescript
// Union simples (string | number)
const stringOrNumber = z.union([z.string(), z.number()]);
stringOrNumber.parse("hello"); // OK
stringOrNumber.parse(42); // OK
stringOrNumber.parse(true); // Erro

// Alternativa com .or()
const stringOrNumberAlt = z.string().or(z.number());

// Discriminated unions
const userShape = z.object({
  type: z.literal("user"),
  name: z.string()
});

const adminShape = z.object({
  type: z.literal("admin"),
  name: z.string(),
  permissions: z.array(z.string())
});

const personUnion = z.discriminatedUnion("type", [
  userShape,
  adminShape
]);

// Exemplos válidos:
personUnion.parse({
  type: "user",
  name: "John"
});

personUnion.parse({
  type: "admin",
  name: "Jane",
  permissions: ["read", "write"]
});

// Inválido (tipo não corresponde):
personUnion.parse({
  type: "guest",
  name: "Bob"
});

// Inválido (campos incompatíveis):
personUnion.parse({
  type: "user",
  name: "Alice",
  permissions: ["read"] // campo extra para type="user"
});
```

### Intersection Types

```typescript
// Combinando duas formas usando interseção
const basePerson = z.object({
  name: z.string(),
  age: z.number()
});

const withEmail = z.object({
  email: z.string().email()
});

// Interseção (AND lógico)
const personWithEmail = z.intersection(
  basePerson,
  withEmail
);

// Alternativa com .and()
const personWithEmailAlt = basePerson.and(withEmail);

// Exemplo de uso:
personWithEmail.parse({
  name: "Carlos",
  age: 35,
  email: "carlos@example.com"
});
```

## Inferência de Tipos

```typescript
// Inferindo tipos TypeScript do schema Zod
const userSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email().optional(),
  role: z.enum(["admin", "user", "guest"]),
  metadata: z.record(z.string()) // objeto de strings
});

// Extrair o tipo TypeScript
type User = z.infer<typeof userSchema>;

// Agora o tipo User é equivalente a:
// type User = {
//   id: number;
//   name: string;
//   email?: string | undefined;
//   role: "admin" | "user" | "guest";
//   metadata: Record<string, string>;
// }

// Uso prático:
function createUser(userData: User) {
  // Tipo já está validado em tempo de compilação
  return userData;
}

// Exemplo com Array
const todoSchema = z.array(
  z.object({
    id: z.number(),
    task: z.string(),
    completed: z.boolean()
  })
);

type TodoList = z.infer<typeof todoSchema>;
// type TodoList = {
//   id: number;
//   task: string;
//   completed: boolean;
// }[]
```

## Transformações

```typescript
// Transformação básica de string para número
const stringToNumber = z.string()
  .transform(str => parseInt(str, 10));

stringToNumber.parse("42"); // Retorna 42 (number)

// Sequência de transformações
const urlToHostname = z.string().url()
  .transform(url => new URL(url))
  .transform(url => url.hostname);

urlToHostname.parse("https://example.com/path"); // Retorna "example.com"

// Transformando dados de form
const formSchema = z.object({
  name: z.string(),
  birthYear: z.string()
    .regex(/^\d{4}$/)
    .transform(val => parseInt(val, 10)),
  newsletter: z.string()
    .transform(val => val === "yes")
})
.transform(data => ({
  ...data,
  age: new Date().getFullYear() - data.birthYear
}));

const result = formSchema.parse({
  name: "Ana",
  birthYear: "1990",
  newsletter: "yes"
});
// result = { name: "Ana", birthYear: 1990, newsletter: true, age: 35 }
```

### Transformações Assíncronas

```typescript
// Transformação assíncrona para verificar se um usuário existe
const usernameSchema = z.string()
  .min(3)
  .refine(
    async (username) => {
      // Simulando uma verificação em banco de dados
      return new Promise(resolve => {
        setTimeout(() => {
          const userExists = username !== "admin";
          resolve(userExists);
        }, 100);
      });
    },
    { message: "Nome de usuário já está em uso" }
  );

// Para usar transformações assíncronas, use parseAsync
async function validateUsername(username: string) {
  try {
    await usernameSchema.parseAsync(username);
    return "Nome de usuário disponível";
  } catch (error) {
    return error.errors[0];
  }
}
```

## Validações Assíncronas

```typescript
// Schema com validação assíncrona
const emailSchema = z.string().email().refine(
  async (email) => {
    // Simulando validação de email (verificação de existência)
    return new Promise(resolve => {
      setTimeout(() => {
        // Lógica de validação (exemplo simplificado)
        const invalidDomains = ["example.com", "test.com"];
        const domain = email.split("@")[1];
        resolve(!invalidDomains.includes(domain));
      }, 200);
    });
  },
  { message: "Este domínio de email não é permitido" }
);

// Usando o schema assíncrono
async function validateEmail(email: string) {
  try {
    await emailSchema.parseAsync(email);
    return "Email válido!";
  } catch (error) {
    return `Email inválido: ${error.errors[0]}`;
  }
}

// Schema com múltiplas validações assíncronas
const registerSchema = z.object({
  username: z.string().min(3),
  email: z.string().email(),
}).superRefine(async (data, ctx) => {
  // Verifica se username já existe
  const usernameExists = await checkUsernameInDB(data.username);
  if (usernameExists) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: "Username já está em uso",
      path: ["username"]
    });
  }
  
  // Verifica se email já existe
  const emailExists = await checkEmailInDB(data.email);
  if (emailExists) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: "Email já está registrado",
      path: ["email"]
    });
  }
});

// Funções auxiliares simuladas
async function checkUsernameInDB(username: string) {
  return new Promise(resolve => {
    setTimeout(() => resolve(username === "admin"), 100);
  });
}

async function checkEmailInDB(email: string) {
  return new Promise(resolve => {
    setTimeout(() => resolve(email === "admin@example.com"), 100);
  });
}
```

## Integrações com React e Form Handling

### React Hook Form + Zod

```typescript
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

// Define o schema de validação
const signupSchema = z.object({
  username: z.string()
    .min(3, "Username deve ter pelo menos 3 caracteres")
    .max(20, "Username deve ter no máximo 20 caracteres"),
  email: z.string()
    .email("Email inválido"),
  password: z.string()
    .min(8, "Senha deve ter pelo menos 8 caracteres")
    .refine(
      password => /[A-Z]/.test(password),
      "Senha deve conter pelo menos uma letra maiúscula"
    ),
  confirmPassword: z.string()
}).refine(
  data => data.password === data.confirmPassword,
  {
    message: "As senhas não coincidem",
    path: ["confirmPassword"]
  }
);

// Criar o tipo a partir do schema
type SignupFormData = z.infer<typeof signupSchema>;

// Componente React
function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors }
  } = useForm<SignupFormData>({
    resolver: zodResolver(signupSchema)
  });

  const onSubmit = (data: SignupFormData) => {
    console.log("Form data:", data);
    // Enviar para API...
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Username</label>
        <input {...register("username")} />
        {errors.username && <p>{errors.username.message}</p>}
      </div>
      
      <div>
        <label>Email</label>
        <input {...register("email")} />
        {errors.email && <p>{errors.email.message}</p>}
      </div>
      
      <div>
        <label>Password</label>
        <input type="password" {...register("password")} />
        {errors.password && <p>{errors.password.message}</p>}
      </div>
      
      <div>
        <label>Confirm Password</label>
        <input type="password" {...register("confirmPassword")} />
        {errors.confirmPassword && <p>{errors.confirmPassword.message}</p>}
      </div>
      
      <button type="submit">Cadastrar</button>
    </form>
  );
}
```

### Formulário sem biblioteca (uso manual do Zod)

```typescript
import React, { useState } from "react";
import { z } from "zod";

const contactSchema = z.object({
  name: z.string().min(2, "Nome deve ter pelo menos 2 caracteres"),
  email: z.string().email("Email inválido"),
  message: z.string().min(10, "Mensagem deve ter pelo menos 10 caracteres")
});

type ContactFormData = z.infer<typeof contactSchema>;

function ContactForm() {
  const [formData, setFormData] = useState<Partial<ContactFormData>>({
    name: "",
    email: "",
    message: ""
  });
  
  const [errors, setErrors] = useState<Record<string, string>>({});
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    try {
      // Validar dados com Zod
      const validData = contactSchema.parse(formData);
      console.log("Dados válidos:", validData);
      
      // Limpar erros e processar o envio
      setErrors({});
      // Enviar para API...
      
    } catch (error) {
      if (error instanceof z.ZodError) {
        // Converter erros do Zod para formato de objeto
        const fieldErrors: Record<string, string> = {};
        error.errors.forEach(err => {
          const path = err.path.join(".");
          fieldErrors[path] = err.message;
        });
        setErrors(fieldErrors);
      }
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Nome</label>
        <input
          name="name"
          value={formData.name || ""}
          onChange={handleChange}
        />
        {errors.name && <p>{errors.name}</p>}
      </div>
      
      <div>
        <label>Email</label>
        <input
          name="email"
          value={formData.email || ""}
          onChange={handleChange}
        />
        {errors.email && <p>{errors.email}</p>}
      </div>
      
      <div>
        <label>Mensagem</label>
        <textarea
          name="message"
          value={formData.message || ""}
          onChange={handleChange}
        />
        {errors.message && <p>{errors.message}</p>}
      </div>
      
      <button type="submit">Enviar</button>
    </form>
  );
}
```

## Casos de Uso Avançados

### Validação de API

```typescript
import { z } from "zod";

// Schema para validar resposta da API
const userResponseSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  posts: z.array(
    z.object({
      id: z.number(),
      title: z.string(),
      body: z.string()
    })
  )
});

// Tipo inferido da resposta
type UserResponse = z.infer<typeof userResponseSchema>;

// Função para buscar dados com validação de tipos
async function fetchUser(userId: number): Promise<UserResponse> {
  try {
    const response = await fetch(`https://api.example.com/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`Erro HTTP: ${response.status}`);
    }
    
    const data = await response.json();
    
    // Validar dados recebidos
    return userResponseSchema.parse(data);
  } catch (error) {
    if (error instanceof z.ZodError) {
      console.error("Erro de validação:", error.errors);
      throw new Error("A resposta da API não está no formato esperado");
    }
    throw error;
  }
}

// Uso:
fetchUser(1)
  .then(user => console.log("Usuário validado:", user))
  .catch(error => console.error("Erro:", error));
```

### Enum Dinâmico

```typescript
import { z } from "zod";

// Simula busca de roles do backend
async function fetchRoles() {
  // Simulação de uma chamada API
  return new Promise<string[]>(resolve => {
    setTimeout(() => {
      resolve(["admin", "editor", "viewer", "support"]);
    }, 500);
  });
}

// Função para criar schema com enum dinâmico
async function createUserSchema() {
  const roles = await fetchRoles();
  
  return z.object({
    name: z.string(),
    email: z.string().email(),
    role: z.enum(roles as [string, ...string[]])
  });
}

// Uso:
async function validateUser(userData: unknown) {
  const schema = await createUserSchema();
  
  try {
    const validUser = schema.parse(userData);
    console.log("Usuário válido:", validUser);
    return validUser;
  } catch (error) {
    console.error("Erro de validação:", error);
    throw error;
  }
}

// Exemplo
const user = {
  name: "Manuel",
  email: "manuel@example.com",
  role: "editor"
};

validateUser(user);
```

### Sistema de Validação Plugável

```typescript
import { z } from "zod";

// Esquema base
const baseProductSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(3),
  price: z.number().positive(),
  stock: z.number().int().nonnegative()
});

// Plugins de validação por país
const validationPlugins = {
  BR: (schema: typeof baseProductSchema) => schema.extend({
    price: z.number().positive().refine(
      price => price > 0.1,
      { message: "Preço mínimo deve ser maior que R$ 0,10" }
    ),
    tax: z.number().min(0).max(0.4)
  }),
  
  US: (schema: typeof baseProductSchema) => schema.extend({
    price: z.number().positive().refine(
      price => price > 0.01,
      { message: "Preço mínimo deve ser maior que $0.01" }
    ),
    shippingInfo: z.object({
      domestic: z.boolean(),
      estimatedDays: z.number().int().positive()
    })
  }),
  
  EU: (schema: typeof baseProductSchema) => schema.extend({
    price: z.number().positive(),
    vatIncluded: z.boolean(),
    vatPercentage: z.number().min(0).max(0.27)
  })
};

// Função para obter o schema específico para um país
function getProductSchemaForCountry(countryCode: keyof typeof validationPlugins) {
  const plugin = validationPlugins[countryCode];
  if (!plugin) {
    return baseProductSchema; // Schema padrão se não tiver plugin
  }
  return plugin(baseProductSchema);
}

// Exemplos de uso:
try {
  // Validar produto para o Brasil
  const brSchema = getProductSchemaForCountry("BR");
  const validBrProduct = brSchema.parse({
    id: "123e4567-e89b-12d3-a456-426614174000",
    name: "Produto Teste",
    price: 29.99,
    stock: 100,
    tax: 0.09 // Campo específico para BR
  });
  console.log("Produto BR válido:", validBrProduct);
  
  // Validar produto para os EUA
  const usSchema = getProductSchemaForCountry("US");
  const validUsProduct = usSchema.parse({
    id: "123e4567-e89b-12d3-a456-426614174000",
    name: "Test Product",
    price: 19.99,
    stock: 50,
    shippingInfo: {
      domestic: true,
      estimatedDays: 3
    }
  });
  console.log("Produto US válido:", validUsProduct);
} catch (error) {
  console.error("Erro de validação:", error);
}
```

### Schema Builder Pattern

```typescript
import { z } from "zod";

// Builder para criar schemas de forma mais flexível
class SchemaBuilder<T extends z.ZodTypeAny> {
  private schema: T;
  
  constructor(initialSchema: T) {
    this.schema = initialSchema;
  }
  
  withEmail(options?: { required?: boolean }) {
    const emailSchema = z.string().email("Email inválido");
    const finalSchema = options?.required === false
      ? emailSchema.optional()
      : emailSchema;
      
    return this.extend({ email: finalSchema });
  }
  
  withPassword(minLength = 8) {
    return this.extend({
      password: z.string().min(minLength, `Senha deve ter pelo menos ${minLength} caracteres`)
    });
  }
  
  withAge(options?: { min?: number; max?: number }) {
    let ageSchema = z.number().int("Idade deve ser um número inteiro");
    
    if (options?.min !== undefined) {
      ageSchema = ageSchema.min(options.min, `Idade mínima é ${options.min}`);
    }
    
    if (options?.max !== undefined) {
      ageSchema = ageSchema.max(options.max, `Idade máxima é ${options.max}`);
    }
    
    return this.extend({ age: ageSchema });
  }
  
  withAddress(required = true) {
    const addressSchema = z.object({
      street: z.string(),
      city: z.string(),
      state: z.string(),
      zipCode: z.string()
    });
    
    return this.extend({ 
      address: required ? addressSchema : addressSchema.optional() 
    });
  }
  
  private extend<K extends string, V extends z.ZodTypeAny>(
    fields: Record<K, V>
  ) {
    // Se for um objeto, estendemos com os novos campos
    if (this.schema instanceof z.ZodObject) {
      const newSchema = this.schema.extend(fields as any);
      return new SchemaBuilder(newSchema);
    }
    
    // Caso não seja um objeto, começamos um novo
    const newSchema = z.object(fields as any);
    return new SchemaBuilder(newSchema);
  }
  
  // Métodos auxiliares
  optional() {
    return new SchemaBuilder(this.schema.optional());
  }
  
  build() {
    return this.schema;
  }
}

// Criar um builder inicial com um objeto vazio
const createSchema = () => new SchemaBuilder(z.object({}));

// Exemplo de uso do builder
const userSchema = createSchema()
  .withEmail()
  .withPassword(10)
  .withAge({ min: 18 })
  .build();

// Tipo inferido do schema
type User = z.infer<typeof userSchema>;

// Exemplo de validação
try {
  const validUser = userSchema.parse({
    email: "user@example.com",
    password: "securepass123",
    age: 25
  });
  console.log("Usuário válido:", validUser);
} catch (error) {
  console.error("Erro de validação:", error);
}

// Outro exemplo com campos diferentes
const customerSchema = createSchema()
  .withEmail()
  .withAddress()
  .build();

// Validação do customer
try {
  const validCustomer = customerSchema.parse({
    email: "customer@example.com",
    address: {
      street: "Rua Principal, 123",
      city: "São Paulo",
      state: "SP",
      zipCode: "01234-567"
    }
  });
  console.log("Cliente válido:", validCustomer);
} catch (error) {
  console.error("Erro de validação:", error);
}
```

### Validação em Camadas

```typescript
import { z } from "zod";

// Schema Base (Dados Brutos)
const rawUserDataSchema = z.object({
  name: z.string(),
  email: z.string(),
  birthDate: z.string(),
  roles: z.array(z.string())
});

// Camada 1: Validação Básica (Formato)
const validatedUserSchema = rawUserDataSchema
  .refine(
    data => data.email.includes("@"),
    { path: ["email"], message: "Email deve conter @" }
  )
  .refine(
    data => /^\d{4}-\d{2}-\d{2}$/.test(data.birthDate),
    { path: ["birthDate"], message: "Data de nascimento deve estar no formato YYYY-MM-DD" }
  );

// Camada 2: Transformação para Modelo de Domínio
const domainUserSchema = validatedUserSchema
  .transform(data => ({
    ...data,
    email: data.email.toLowerCase(),
    birthDate: new Date(data.birthDate),
    roles: [...new Set(data.roles)], // Remove duplicados
    age: new Date().getFullYear() - new Date(data.birthDate).getFullYear()
  }));

// Camada 3: Validação de Regras de Negócio
const businessRulesUserSchema = domainUserSchema
  .refine(
    data => data.age >= 18,
    { message: "Usuário deve ter pelo menos 18 anos" }
  )
  .refine(
    data => {
      const allowedRoles = ["admin", "user", "editor", "guest"];
      return data.roles.every(role => allowedRoles.includes(role));
    },
    { path: ["roles"], message: "Função não permitida" }
  );

// Uso do pipeline de validação
async function processUser(rawData: unknown) {
  try {
    // Passando pelos vários níveis de validação e transformação
    const rawValidated = rawUserDataSchema.parse(rawData);
    console.log("1. Dados brutos validados:", rawValidated);
    
    const formatValidated = validatedUserSchema.parse(rawValidated);
    console.log("2. Formato validado:", formatValidated);
    
    const domainUser = domainUserSchema.parse(formatValidated);
    console.log("3. Transformado para modelo de domínio:", domainUser);
    
    const businessValidatedUser = businessRulesUserSchema.parse(domainUser);
    console.log("4. Regras de negócio validadas:", businessValidatedUser);
    
    return businessValidatedUser;
  } catch (error) {
    if (error instanceof z.ZodError) {
      console.error("Erro de validação:", JSON.stringify(error.errors, null, 2));
    }
    throw error;
  }
}

// Exemplo
const userData = {
  name: "Carlos Silva",
  email: "Carlos@example.com",
  birthDate: "1995-08-15",
  roles: ["user", "editor", "editor"] // nota: tem um duplicado
};

processUser(userData)
  .then(user => console.log("Usuário processado com sucesso:", user))
  .catch(error => console.error("Falha ao processar usuário:", error));
```

### Pré-processamento de Dados

```typescript
import { z } from "zod";

// Schema com pré-processamento
const formDataSchema = z.preprocess(
  // Função de pré-processamento para sanitizar dados
  (data: any) => {
    if (typeof data !== 'object' || data === null) {
      return data;
    }
    
    const processed = { ...data };
    
    // Converter strings vazias para undefined
    Object.keys(processed).forEach(key => {
      if (processed[key] === '') {
        processed[key] = undefined;
      }
      
      // Remover espaços extras em strings
      if (typeof processed[key] === 'string') {
        processed[key] = processed[key].trim();
      }
    });
    
    return processed;
  },
  // Schema real que será aplicado após o pré-processamento
  z.object({
    name: z.string().min(2),
    email: z.string().email(),
    age: z.number().int().positive().optional(),
    website: z.string().url().optional(),
    notes: z.string().optional()
  })
);

// Exemplo de dados de formulário com problemas comuns
const rawFormData = {
  name: "  João Silva  ",
  email: "joao@example.com",
  age: "30", // string ao invés de número
  website: "",  // string vazia
  notes: "   " // só espaços
};

// Versão melhorada com coerção de tipos
const betterFormSchema = z.object({
  name: z.preprocess(
    val => typeof val === 'string' ? val.trim() : val,
    z.string().min(2)
  ),
  email: z.preprocess(
    val => typeof val === 'string' ? val.trim().toLowerCase() : val,
    z.string().email()
  ),
  age: z.preprocess(
    val => val === '' ? undefined : isNaN(Number(val)) ? val : Number(val),
    z.number().int().positive().optional()
  ),
  website: z.preprocess(
    val => typeof val === 'string' && val.trim() === '' ? undefined : val,
    z.string().url().optional()
  ),
  notes: z.preprocess(
    val => typeof val === 'string' && val.trim() === '' ? undefined : val,
    z.string().optional()
  )
});

// Processamento de dados de formulário
function processForm(formData: unknown) {
  try {
    const validData = betterFormSchema.parse(formData);
    console.log("Dados do formulário processados:", validData);
    return validData;
  } catch (error) {
    console.error("Erro no processamento do formulário:", error);
    throw error;
  }
}

// Exemplo
processForm(rawFormData);
```

### Middleware Pattern com Zod

```typescript
import { z } from "zod";
import express from "express";

// Função para criar middleware de validação
function validateRequest<T extends z.ZodTypeAny>(
  schema: T,
  source: 'body' | 'query' | 'params' = 'body'
) {
  return (req: express.Request, res: express.Response, next: express.NextFunction) => {
    try {
      const data = schema.parse(req[source]);
      req[source] = data; // Substitui os dados originais pelos validados
      next();
    } catch (error) {
      if (error instanceof z.ZodError) {
        return res.status(400).json({
          status: 'error',
          message: 'Dados de entrada inválidos',
          errors: error.errors
        });
      }
      next(error);
    }
  };
}

// Exemplo de uso em uma aplicação Express
const app = express();
app.use(express.json());

// Schema para criação de usuário
const createUserSchema = z.object({
  name: z.string().min(2, "Nome deve ter pelo menos 2 caracteres"),
  email: z.string().email("Email inválido"),
  password: z.string().min(8, "Senha deve ter pelo menos 8 caracteres"),
  role: z.enum(["admin", "user"], {
    errorMap: () => ({ message: "Função deve ser 'admin' ou 'user'" })
  })
});

// Schema para parâmetros de rota
const userIdSchema = z.object({
  id: z.string().uuid("ID de usuário inválido")
});

// Schema para query string
const userQuerySchema = z.object({
  role: z.enum(["admin", "user"]).optional(),
  active: z.preprocess(
    val => val === 'true' ? true : val === 'false' ? false : val,
    z.boolean().optional()
  ),
  page: z.preprocess(
    val => (val && !isNaN(Number(val))) ? Number(val) : val,
    z.number().int().positive().optional().default(1)
  ),
  limit: z.preprocess(
    val => (val && !isNaN(Number(val))) ? Number(val) : val,
    z.number().int().positive().optional().default(10)
  )
});

// Rotas com validação
app.post(
  '/users',
  validateRequest(createUserSchema),
  (req, res) => {
    // req.body já está validado e tipado
    const user = req.body;
    // Lógica para criar usuário...
    res.status(201).json({ status: 'success', data: user });
  }
);

app.get(
  '/users/:id',
  validateRequest(userIdSchema, 'params'),
  (req, res) => {
    const { id } = req.params;
    // Lógica para buscar usuário por ID...
    res.json({ status: 'success', data: { id, name: 'Exemplo' } });
  }
);

app.get(
  '/users',
  validateRequest(userQuerySchema, 'query'),
  (req, res) => {
    const { role, active, page, limit } = req.query;
    // Lógica para listar usuários com filtros...
    res.json({
      status: 'success',
      data: [],
      pagination: { page, limit, total: 0 }
    });
  }
);

// Tratamento de erros global
app.use((err: any, req: express.Request, res: express.Response, next: express.NextFunction) => {
  console.error(err);
  res.status(500).json({
    status: 'error',
    message: 'Erro interno do servidor'
  });
});

// Iniciar servidor
const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
```

### Composição Avançada de Schemas

```typescript
import { z } from "zod";

// Componentes reutilizáveis
const IdSchema = z.string().uuid();
const TimestampsSchema = z.object({
  createdAt: z.date(),
  updatedAt: z.date()
});

// Função para adicionar campos a um schema existente
function withTimestamps<T extends z.ZodRawShape>(schema: z.ZodObject<T>) {
  return schema.merge(TimestampsSchema);
}

function withId<T extends z.ZodRawShape>(schema: z.ZodObject<T>) {
  return schema.extend({ id: IdSchema });
}

function withSoftDelete<T extends z.ZodRawShape>(schema: z.ZodObject<T>) {
  return schema.extend({ deletedAt: z.date().nullable().optional() });
}

function withVersioning<T extends z.ZodRawShape>(schema: z.ZodObject<T>) {
  return schema.extend({ version: z.number().int().nonnegative().default(1) });
}

// Schema base
const baseTaskSchema = z.object({
  title: z.string().min(3),
  description: z.string().optional(),
  priority: z.enum(["low", "medium", "high"]),
  completed: z.boolean().default(false)
});

// Composição de schemas
const taskSchema = baseTaskSchema.pipe(
  withId(baseTaskSchema)
).pipe(
  withTimestamps(withId(baseTaskSchema))
).pipe(
  withSoftDelete(withTimestamps(withId(baseTaskSchema)))
).pipe(
  withVersioning(withSoftDelete(withTimestamps(withId(baseTaskSchema))))
);

// Tipo resultante
type Task = z.infer<typeof taskSchema>;

// Exemplo de uso
const exampleTask: Task = {
  id: "123e4567-e89b-12d3-a456-426614174000",
  title: "Implementar autenticação",
  description: "Adicionar login com JWT",
  priority: "high",
  completed: false,
  createdAt: new Date(),
  updatedAt: new Date(),
  deletedAt: null,
  version: 1
};

// Validação
const validTask = taskSchema.parse(exampleTask);
console.log("Tarefa válida:", validTask);

// Criação de schemas específicos para diferentes operações
const createTaskSchema = baseTaskSchema.omit({ completed: true });
const updateTaskSchema = baseTaskSchema.partial();
const listTasksResponseSchema = z.array(taskSchema);

// Exemplo de operações
function simulateApi() {
  // Criar tarefa (dados do cliente)
  const newTaskData = {
    title: "Nova tarefa",
    priority: "medium"
  };
  
  // Validar dados de criação
  const validNewTask = createTaskSchema.parse(newTaskData);
  console.log("Dados para criação válidos:", validNewTask);
  
  // Simular resposta da API (já com campos adicionais)
  const createdTask = taskSchema.parse({
    ...validNewTask,
    id: "123e4567-e89b-12d3-a456-426614174000",
    completed: false,
    createdAt: new Date(),
    updatedAt: new Date(),
    deletedAt: null,
    version: 1
  });
  
  // Atualizar tarefa (dados parciais)
  const updateData = {
    completed: true,
    description: "Descrição atualizada"
  };
  
  // Validar dados de atualização
  const validUpdateData = updateTaskSchema.parse(updateData);
  console.log("Dados para atualização válidos:", validUpdateData);
  
  // Simular resposta da API após atualização
  const updatedTask = taskSchema.parse({
    ...createdTask,
    ...validUpdateData,
    updatedAt: new Date(),
    version: 2
  });
  
  return updatedTask;
}

const result = simulateApi();
console.log("Resultado final:", result);
```

## Conclusão

O Zod é uma ferramenta poderosa para validação de dados em TypeScript, oferecendo não apenas segurança de tipos, mas também validação em tempo de execução. Com sua API fluente e extensível, é possível criar esquemas de validação simples a extremamente complexos.

Principais vantagens:

1. **Segurança de tipos**: Integração perfeita com TypeScript
2. **API declarativa**: Fácil de ler e entender
3. **Validações ricas**: Ampla gama de validadores embutidos
4. **Extensibilidade**: Fácil de estender com validações personalizadas
5. **Transformações**: Capacidade de transformar dados durante a validação
6. **Integrações**: Funciona bem com React, Express e outras bibliotecas populares

Lembre-se de sempre tratar erros de validação adequadamente e fornecer mensagens claras para os usuários. O Zod facilita muito esta tarefa, permitindo personalizar mensagens de erro em cada nível.

Para mais informações, consulte a [documentação oficial do Zod](https://github.com/colinhacks/zod).
