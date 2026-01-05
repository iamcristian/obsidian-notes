# NestJS Complete Guide

> **Guía profesional de NestJS 10 para desarrollo y entrevistas técnicas**

---

## 📚 Tabla de Contenidos

1. [[#Fundamentos de NestJS|Fundamentos]]
2. [[#Módulos|Módulos]]
3. [[#Controllers|Controllers]]
4. [[#Providers y Services|Services]]
5. [[#Dependency Injection|DI]]
6. [[#Pipes y Validation|Validation]]
7. [[#Guards|Guards]]
8. [[#Interceptors|Interceptors]]
9. [[#Exception Filters|Filters]]
10. [[#Middleware|Middleware]]

---

## Fundamentos de NestJS

### ¿Qué es NestJS?

NestJS es un **framework de Node.js** para construir aplicaciones server-side eficientes y escalables.

```
┌─────────────────────────────────────────────────────────┐
│                    NESTJS FEATURES                      │
├─────────────────────────────────────────────────────────┤
│  • TypeScript first (también soporta JavaScript)        │
│  • Arquitectura modular inspirada en Angular            │
│  • Dependency Injection incorporado                     │
│  • Decoradores para definir metadata                    │
│  • Soporte para REST, GraphQL, WebSockets, gRPC         │
│  • Testing utilities incluidas                          │
│  • CLI potente para scaffolding                         │
│  • Extensible con módulos de la comunidad               │
└─────────────────────────────────────────────────────────┘
```

### Request Lifecycle

```
┌─────────────────────────────────────────────────────────┐
│                  REQUEST LIFECYCLE                      │
├─────────────────────────────────────────────────────────┤
│  1. Incoming Request                                    │
│  2. Middleware (global → module → route)                │
│  3. Guards (global → controller → route)                │
│  4. Interceptors PRE (global → controller → route)      │
│  5. Pipes (global → controller → route → param)         │
│  6. Controller (route handler)                          │
│  7. Service (business logic)                            │
│  8. Interceptors POST (route → controller → global)     │
│  9. Exception Filters (route → controller → global)     │
│  10. Response                                           │
└─────────────────────────────────────────────────────────┘
```

### Instalación y Setup

```bash
# Instalar CLI
npm install -g @nestjs/cli

# Crear nuevo proyecto
nest new my-project

# Estructura generada:
my-project/
├── src/
│   ├── app.controller.ts      # Controller básico
│   ├── app.controller.spec.ts # Tests del controller
│   ├── app.module.ts          # Módulo raíz
│   ├── app.service.ts         # Service básico
│   └── main.ts                # Entry point
├── test/
│   ├── app.e2e-spec.ts        # Tests E2E
│   └── jest-e2e.json          # Config Jest E2E
├── nest-cli.json              # Config CLI
├── tsconfig.json              # Config TypeScript
└── package.json
```

### Entry Point

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Configuración global
  app.setGlobalPrefix('api/v1');
  
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,           // Elimina propiedades no decoradas
    forbidNonWhitelisted: true, // Error si hay props extra
    transform: true,            // Transforma payloads a DTO instances
    transformOptions: {
      enableImplicitConversion: true,
    },
  }));
  
  app.enableCors({
    origin: process.env.FRONTEND_URL,
    credentials: true,
  });
  
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

---

## Módulos

### Estructura de un Módulo

```typescript
// Los módulos organizan la aplicación en bloques cohesivos

// users/users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { User } from './entities/user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])], // Módulos importados
  controllers: [UsersController],              // Controllers del módulo
  providers: [UsersService],                   // Providers (services, etc.)
  exports: [UsersService],                     // Lo que exporta a otros módulos
})
export class UsersModule {}
```

### Tipos de Módulos

```typescript
// 1. Feature Module - agrupa funcionalidad relacionada
@Module({
  imports: [TypeOrmModule.forFeature([Product, Category])],
  controllers: [ProductsController, CategoriesController],
  providers: [ProductsService, CategoriesService],
  exports: [ProductsService],
})
export class ProductsModule {}

// 2. Shared Module - funcionalidad compartida
@Module({
  providers: [LoggerService, UtilsService],
  exports: [LoggerService, UtilsService],
})
export class SharedModule {}

// 3. Global Module - disponible en toda la app sin importar
@Global()
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule {}

// 4. Dynamic Module - configuración en runtime
@Module({})
export class DatabaseModule {
  static forRoot(options: DatabaseOptions): DynamicModule {
    return {
      module: DatabaseModule,
      providers: [
        {
          provide: 'DATABASE_OPTIONS',
          useValue: options,
        },
        DatabaseService,
      ],
      exports: [DatabaseService],
    };
  }
  
  static forRootAsync(options: DatabaseAsyncOptions): DynamicModule {
    return {
      module: DatabaseModule,
      imports: options.imports || [],
      providers: [
        {
          provide: 'DATABASE_OPTIONS',
          useFactory: options.useFactory,
          inject: options.inject || [],
        },
        DatabaseService,
      ],
      exports: [DatabaseService],
    };
  }
}

// Uso del módulo dinámico
@Module({
  imports: [
    DatabaseModule.forRoot({
      host: 'localhost',
      port: 5432,
    }),
    // O async
    DatabaseModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (config: ConfigService) => ({
        host: config.get('DB_HOST'),
        port: config.get('DB_PORT'),
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

### App Module (Root)

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersModule } from './users/users.module';
import { AuthModule } from './auth/auth.module';
import { ProductsModule } from './products/products.module';

@Module({
  imports: [
    // Configuración global
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: `.env.${process.env.NODE_ENV || 'development'}`,
    }),
    
    // Base de datos
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (config: ConfigService) => ({
        type: 'postgres',
        host: config.get('DB_HOST'),
        port: config.get('DB_PORT'),
        username: config.get('DB_USER'),
        password: config.get('DB_PASSWORD'),
        database: config.get('DB_NAME'),
        autoLoadEntities: true,
        synchronize: config.get('NODE_ENV') !== 'production',
      }),
      inject: [ConfigService],
    }),
    
    // Feature modules
    UsersModule,
    AuthModule,
    ProductsModule,
  ],
})
export class AppModule {}
```

---

## Controllers

### Estructura de Controller

```typescript
// users/users.controller.ts
import {
  Controller,
  Get,
  Post,
  Put,
  Patch,
  Delete,
  Body,
  Param,
  Query,
  Headers,
  Req,
  Res,
  HttpCode,
  HttpStatus,
  ParseIntPipe,
  ParseUUIDPipe,
} from '@nestjs/common';
import { Request, Response } from 'express';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { QueryUsersDto } from './dto/query-users.dto';

@Controller('users') // Prefijo de ruta: /users
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  // GET /users
  @Get()
  findAll(@Query() query: QueryUsersDto) {
    return this.usersService.findAll(query);
  }

  // GET /users/:id
  @Get(':id')
  findOne(@Param('id', ParseUUIDPipe) id: string) {
    return this.usersService.findOne(id);
  }

  // POST /users
  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  // PUT /users/:id (reemplazo completo)
  @Put(':id')
  replace(
    @Param('id', ParseUUIDPipe) id: string,
    @Body() createUserDto: CreateUserDto,
  ) {
    return this.usersService.replace(id, createUserDto);
  }

  // PATCH /users/:id (actualización parcial)
  @Patch(':id')
  update(
    @Param('id', ParseUUIDPipe) id: string,
    @Body() updateUserDto: UpdateUserDto,
  ) {
    return this.usersService.update(id, updateUserDto);
  }

  // DELETE /users/:id
  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id', ParseUUIDPipe) id: string) {
    return this.usersService.remove(id);
  }

  // Acceder al request/response de Express
  @Get('custom')
  customResponse(@Req() req: Request, @Res() res: Response) {
    res.status(200).json({ custom: true });
  }

  // Headers personalizados
  @Get('with-headers')
  getWithHeaders(@Headers('authorization') auth: string) {
    return { auth };
  }
}
```

### Decoradores de Parámetros

```typescript
@Controller('products')
export class ProductsController {
  // @Param - parámetros de ruta
  @Get(':category/:id')
  findByCategory(
    @Param('category') category: string,
    @Param('id', ParseIntPipe) id: number,
  ) {
    return { category, id };
  }

  // @Query - query parameters
  @Get()
  findAll(
    @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
    @Query('limit', new DefaultValuePipe(10), ParseIntPipe) limit: number,
    @Query('search') search?: string,
  ) {
    return { page, limit, search };
  }

  // @Body - cuerpo de la petición
  @Post()
  create(@Body() dto: CreateProductDto) {
    return dto;
  }

  // @Body con propiedad específica
  @Post('partial')
  createPartial(@Body('name') name: string) {
    return { name };
  }

  // @Headers - cabeceras
  @Get('headers')
  getHeaders(
    @Headers() headers: Record<string, string>,
    @Headers('user-agent') userAgent: string,
  ) {
    return { headers, userAgent };
  }

  // @Ip - IP del cliente
  @Get('ip')
  getIp(@Ip() ip: string) {
    return { ip };
  }

  // @HostParam - para subdominios
  @Get()
  @HostParam('account')
  getByAccount(@HostParam('account') account: string) {
    return { account };
  }
}
```

### Rutas Avanzadas

```typescript
// Rutas con wildcards y regex
@Controller('files')
export class FilesController {
  // Wildcard: /files/any/path/here
  @Get('*')
  findAll() {
    return 'Wildcard route';
  }

  // Regex en parámetros
  @Get(':id(\\d+)') // Solo números
  findOne(@Param('id') id: string) {
    return { id };
  }
}

// Sub-recursos
@Controller('users/:userId/posts')
export class UserPostsController {
  @Get()
  findAll(@Param('userId', ParseUUIDPipe) userId: string) {
    return this.postsService.findByUser(userId);
  }

  @Get(':postId')
  findOne(
    @Param('userId', ParseUUIDPipe) userId: string,
    @Param('postId', ParseUUIDPipe) postId: string,
  ) {
    return this.postsService.findOneByUser(userId, postId);
  }
}

// Versionado de API
@Controller({
  path: 'users',
  version: '1',
})
export class UsersV1Controller {
  @Get()
  findAll() {
    return 'V1 users';
  }
}

@Controller({
  path: 'users',
  version: '2',
})
export class UsersV2Controller {
  @Get()
  findAll() {
    return 'V2 users with more data';
  }
}
```

---

## Providers y Services

### Service Básico

```typescript
// users/users.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { QueryUsersDto } from './dto/query-users.dto';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly usersRepository: Repository<User>,
  ) {}

  async findAll(query: QueryUsersDto) {
    const { page = 1, limit = 10, search, sortBy = 'createdAt', order = 'DESC' } = query;
    
    const queryBuilder = this.usersRepository.createQueryBuilder('user');
    
    if (search) {
      queryBuilder.where(
        'user.name ILIKE :search OR user.email ILIKE :search',
        { search: `%${search}%` },
      );
    }
    
    const [users, total] = await queryBuilder
      .orderBy(`user.${sortBy}`, order)
      .skip((page - 1) * limit)
      .take(limit)
      .getManyAndCount();
    
    return {
      data: users,
      meta: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit),
      },
    };
  }

  async findOne(id: string): Promise<User> {
    const user = await this.usersRepository.findOne({
      where: { id },
      relations: ['posts', 'profile'],
    });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return user;
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.usersRepository.findOne({ where: { email } });
  }

  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.usersRepository.create(createUserDto);
    return this.usersRepository.save(user);
  }

  async update(id: string, updateUserDto: UpdateUserDto): Promise<User> {
    const user = await this.findOne(id);
    Object.assign(user, updateUserDto);
    return this.usersRepository.save(user);
  }

  async remove(id: string): Promise<void> {
    const result = await this.usersRepository.delete(id);
    
    if (result.affected === 0) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
  }
}
```

### Tipos de Providers

```typescript
// 1. Class Provider (más común)
@Module({
  providers: [UsersService], // Shorthand para:
  // providers: [{ provide: UsersService, useClass: UsersService }]
})
export class UsersModule {}

// 2. Value Provider - valores estáticos
@Module({
  providers: [
    {
      provide: 'CONFIG',
      useValue: {
        apiUrl: 'https://api.example.com',
        timeout: 5000,
      },
    },
  ],
})
export class AppModule {}

// Uso
@Injectable()
export class ApiService {
  constructor(@Inject('CONFIG') private config: { apiUrl: string; timeout: number }) {}
}

// 3. Factory Provider - creación dinámica
@Module({
  providers: [
    {
      provide: 'ASYNC_CONNECTION',
      useFactory: async (configService: ConfigService) => {
        const connection = await createConnection({
          host: configService.get('DB_HOST'),
        });
        return connection;
      },
      inject: [ConfigService],
    },
  ],
})
export class DatabaseModule {}

// 4. Alias Provider - alias para otro provider
@Module({
  providers: [
    UsersService,
    {
      provide: 'AliasedUsersService',
      useExisting: UsersService,
    },
  ],
})
export class UsersModule {}

// 5. Non-class-based provider tokens
export const DATABASE_CONNECTION = Symbol('DATABASE_CONNECTION');

@Module({
  providers: [
    {
      provide: DATABASE_CONNECTION,
      useFactory: () => createDatabaseConnection(),
    },
  ],
})
export class DatabaseModule {}
```

---

## Dependency Injection

### Inyección por Constructor

```typescript
// La forma más común y recomendada
@Injectable()
export class OrdersService {
  constructor(
    private readonly usersService: UsersService,
    private readonly productsService: ProductsService,
    private readonly emailService: EmailService,
  ) {}

  async createOrder(userId: string, productIds: string[]) {
    const user = await this.usersService.findOne(userId);
    const products = await this.productsService.findByIds(productIds);
    
    const order = await this.saveOrder(user, products);
    
    await this.emailService.sendOrderConfirmation(user.email, order);
    
    return order;
  }
}
```

### Inyección por Propiedad

```typescript
// Útil cuando no puedes modificar el constructor
@Injectable()
export class OrdersService {
  @Inject(UsersService)
  private readonly usersService: UsersService;

  @Inject('CONFIG')
  private readonly config: AppConfig;
}
```

### Scopes de Providers

```typescript
// 1. SINGLETON (default) - una instancia para toda la app
@Injectable() // Equivalente a @Injectable({ scope: Scope.DEFAULT })
export class SingletonService {}

// 2. REQUEST - nueva instancia por cada request
@Injectable({ scope: Scope.REQUEST })
export class RequestScopedService {
  constructor(@Inject(REQUEST) private request: Request) {
    // Acceso al request actual
  }
}

// 3. TRANSIENT - nueva instancia cada vez que se inyecta
@Injectable({ scope: Scope.TRANSIENT })
export class TransientService {
  private readonly id = Math.random();
}

// Propagación de scope
// Si A (singleton) depende de B (request-scoped), A se vuelve request-scoped

// Durable providers - mantienen su instancia entre requests (útil con tenants)
@Injectable({ scope: Scope.REQUEST, durable: true })
export class TenantService {
  constructor(
    @Inject(REQUEST) private request: Request,
    @Inject(CONTEXT_ID_FACTORY) private contextIdFactory: ContextIdFactory,
  ) {}
}
```

### Inyección Circular

```typescript
// Cuando A depende de B y B depende de A
// Usar forwardRef()

// users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @Inject(forwardRef(() => PostsService))
    private readonly postsService: PostsService,
  ) {}
}

// posts.service.ts
@Injectable()
export class PostsService {
  constructor(
    @Inject(forwardRef(() => UsersService))
    private readonly usersService: UsersService,
  ) {}
}

// También en módulos
@Module({
  imports: [forwardRef(() => PostsModule)],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

### Optional Dependencies

```typescript
@Injectable()
export class HttpService {
  constructor(
    @Optional() @Inject('HTTP_OPTIONS')
    private readonly options?: HttpOptions,
  ) {
    // options puede ser undefined
    this.timeout = options?.timeout ?? 5000;
  }
}
```

---

## Pipes y Validation

### DTOs con class-validator

```typescript
// users/dto/create-user.dto.ts
import {
  IsString,
  IsEmail,
  IsNotEmpty,
  MinLength,
  MaxLength,
  IsOptional,
  IsEnum,
  IsArray,
  ValidateNested,
  IsDateString,
  Matches,
  IsStrongPassword,
} from 'class-validator';
import { Type, Transform } from 'class-transformer';

export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator',
}

class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;

  @IsString()
  @Matches(/^\d{5}(-\d{4})?$/, { message: 'Invalid zip code format' })
  zipCode: string;
}

export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  @MinLength(2)
  @MaxLength(50)
  name: string;

  @IsEmail({}, { message: 'Please provide a valid email' })
  @Transform(({ value }) => value.toLowerCase().trim())
  email: string;

  @IsStrongPassword({
    minLength: 8,
    minLowercase: 1,
    minUppercase: 1,
    minNumbers: 1,
    minSymbols: 1,
  })
  password: string;

  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole = UserRole.USER;

  @IsOptional()
  @IsDateString()
  birthDate?: string;

  @IsOptional()
  @IsArray()
  @IsString({ each: true })
  tags?: string[];

  @IsOptional()
  @ValidateNested()
  @Type(() => AddressDto)
  address?: AddressDto;
}

// users/dto/update-user.dto.ts
import { PartialType, OmitType, PickType, IntersectionType } from '@nestjs/mapped-types';
import { CreateUserDto } from './create-user.dto';

// Todos los campos opcionales
export class UpdateUserDto extends PartialType(CreateUserDto) {}

// Omitir campos
export class UpdateProfileDto extends OmitType(CreateUserDto, ['password', 'role']) {}

// Solo ciertos campos
export class UpdatePasswordDto extends PickType(CreateUserDto, ['password']) {}

// Combinar DTOs
export class CreateAdminDto extends IntersectionType(
  CreateUserDto,
  AdditionalAdminFieldsDto,
) {}
```

### Query DTOs

```typescript
// common/dto/pagination.dto.ts
import { IsOptional, IsInt, Min, Max, IsIn } from 'class-validator';
import { Type } from 'class-transformer';

export class PaginationDto {
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 10;

  @IsOptional()
  @IsIn(['ASC', 'DESC'])
  order?: 'ASC' | 'DESC' = 'DESC';
}

// users/dto/query-users.dto.ts
import { IntersectionType } from '@nestjs/mapped-types';
import { IsOptional, IsString, IsEnum } from 'class-validator';
import { PaginationDto } from '../../common/dto/pagination.dto';

export class QueryUsersDto extends PaginationDto {
  @IsOptional()
  @IsString()
  search?: string;

  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;

  @IsOptional()
  @IsString()
  sortBy?: string = 'createdAt';
}
```

### Custom Pipes

```typescript
// common/pipes/parse-date.pipe.ts
import { PipeTransform, Injectable, BadRequestException } from '@nestjs/common';

@Injectable()
export class ParseDatePipe implements PipeTransform<string, Date> {
  transform(value: string): Date {
    const date = new Date(value);
    
    if (isNaN(date.getTime())) {
      throw new BadRequestException(`Invalid date format: ${value}`);
    }
    
    return date;
  }
}

// Uso
@Get('by-date/:date')
findByDate(@Param('date', ParseDatePipe) date: Date) {
  return this.service.findByDate(date);
}

// Pipe con opciones
@Injectable()
export class ParseArrayPipe implements PipeTransform {
  constructor(private readonly options: { separator?: string; type?: 'number' | 'string' }) {}

  transform(value: string): (string | number)[] {
    const separator = this.options.separator ?? ',';
    const items = value.split(separator);
    
    if (this.options.type === 'number') {
      return items.map(item => {
        const num = Number(item);
        if (isNaN(num)) throw new BadRequestException(`Invalid number: ${item}`);
        return num;
      });
    }
    
    return items;
  }
}

// Uso
@Get('by-ids')
findByIds(@Query('ids', new ParseArrayPipe({ type: 'number' })) ids: number[]) {
  return this.service.findByIds(ids);
}
```

### Validation Pipe Avanzado

```typescript
// main.ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,              // Elimina propiedades no decoradas
  forbidNonWhitelisted: true,   // Error si hay props extra
  transform: true,              // Transforma tipos automáticamente
  transformOptions: {
    enableImplicitConversion: true, // Convierte strings a números, etc.
  },
  disableErrorMessages: process.env.NODE_ENV === 'production',
  exceptionFactory: (errors) => {
    const messages = errors.map(error => ({
      field: error.property,
      constraints: Object.values(error.constraints || {}),
    }));
    return new BadRequestException({ errors: messages });
  },
}));

// O por controlador/ruta específica
@Post()
@UsePipes(new ValidationPipe({ transform: true }))
create(@Body() dto: CreateUserDto) {}
```

---

## Guards

### Auth Guard con JWT

```typescript
// auth/guards/jwt-auth.guard.ts
import { Injectable, ExecutionContext, UnauthorizedException } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from '../decorators/public.decorator';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    // Verificar si la ruta es pública
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (isPublic) {
      return true;
    }
    
    return super.canActivate(context);
  }

  handleRequest(err: any, user: any, info: any) {
    if (err || !user) {
      throw err || new UnauthorizedException('Invalid or expired token');
    }
    return user;
  }
}

// auth/decorators/public.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// Uso
@Controller('auth')
export class AuthController {
  @Public()
  @Post('login')
  login(@Body() dto: LoginDto) {}
  
  @Get('profile') // Protegida por defecto
  getProfile(@CurrentUser() user: User) {}
}
```

### Roles Guard

```typescript
// auth/guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY } from '../decorators/roles.decorator';
import { UserRole } from '../../users/enums/user-role.enum';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<UserRole[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (!requiredRoles) {
      return true;
    }
    
    const { user } = context.switchToHttp().getRequest();
    
    return requiredRoles.some(role => user.roles?.includes(role));
  }
}

// auth/decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';
import { UserRole } from '../../users/enums/user-role.enum';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: UserRole[]) => SetMetadata(ROLES_KEY, roles);

// Uso
@Controller('admin')
@Roles(UserRole.ADMIN)
@UseGuards(JwtAuthGuard, RolesGuard)
export class AdminController {
  @Get('dashboard')
  getDashboard() {}
  
  @Roles(UserRole.ADMIN, UserRole.MODERATOR) // Override a nivel de ruta
  @Get('users')
  getUsers() {}
}
```

### Throttle Guard (Rate Limiting)

```typescript
// Instalar: npm install @nestjs/throttler

// app.module.ts
import { ThrottlerModule, ThrottlerGuard } from '@nestjs/throttler';
import { APP_GUARD } from '@nestjs/core';

@Module({
  imports: [
    ThrottlerModule.forRoot([
      {
        name: 'short',
        ttl: 1000,    // 1 segundo
        limit: 3,     // 3 requests
      },
      {
        name: 'medium',
        ttl: 10000,   // 10 segundos
        limit: 20,    // 20 requests
      },
      {
        name: 'long',
        ttl: 60000,   // 1 minuto
        limit: 100,   // 100 requests
      },
    ]),
  ],
  providers: [
    {
      provide: APP_GUARD,
      useClass: ThrottlerGuard,
    },
  ],
})
export class AppModule {}

// Decoradores para override
@Controller('auth')
export class AuthController {
  @Throttle({ default: { limit: 5, ttl: 60000 } }) // Override específico
  @Post('login')
  login() {}
  
  @SkipThrottle() // Sin rate limiting
  @Get('status')
  status() {}
}
```

---

## Interceptors

### Logging Interceptor

```typescript
// common/interceptors/logging.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  Logger,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url, body } = request;
    const now = Date.now();

    this.logger.log(`[${method}] ${url} - Request`);
    this.logger.debug(`Body: ${JSON.stringify(body)}`);

    return next.handle().pipe(
      tap({
        next: (data) => {
          this.logger.log(`[${method}] ${url} - ${Date.now() - now}ms`);
        },
        error: (error) => {
          this.logger.error(`[${method}] ${url} - Error: ${error.message}`);
        },
      }),
    );
  }
}
```

### Transform Interceptor

```typescript
// common/interceptors/transform.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Response<T> {
  success: boolean;
  data: T;
  timestamp: string;
  path: string;
}

@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    const request = context.switchToHttp().getRequest();
    
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
        path: request.url,
      })),
    );
  }
}

// Resultado:
// {
//   "success": true,
//   "data": { ... },
//   "timestamp": "2024-01-15T10:30:00.000Z",
//   "path": "/api/users"
// }
```

### Cache Interceptor

```typescript
// common/interceptors/cache.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable, of } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  private cache = new Map<string, { data: any; expiry: number }>();

  constructor(private readonly ttlSeconds: number = 60) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    // Solo cachear GET requests
    if (request.method !== 'GET') {
      return next.handle();
    }

    const key = this.generateCacheKey(request);
    const cached = this.cache.get(key);

    if (cached && cached.expiry > Date.now()) {
      return of(cached.data);
    }

    return next.handle().pipe(
      tap(data => {
        this.cache.set(key, {
          data,
          expiry: Date.now() + this.ttlSeconds * 1000,
        });
      }),
    );
  }

  private generateCacheKey(request: any): string {
    return `${request.url}-${JSON.stringify(request.query)}`;
  }
}
```

### Timeout Interceptor

```typescript
// common/interceptors/timeout.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  RequestTimeoutException,
} from '@nestjs/common';
import { Observable, throwError, TimeoutError } from 'rxjs';
import { catchError, timeout } from 'rxjs/operators';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  constructor(private readonly timeoutMs: number = 5000) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(this.timeoutMs),
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutException('Request timed out'));
        }
        return throwError(() => err);
      }),
    );
  }
}
```

---

## Exception Filters

### Global Exception Filter

```typescript
// common/filters/all-exceptions.filter.ts
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
  Logger,
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private readonly logger = new Logger(AllExceptionsFilter.name);

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal server error';
    let errors: any = null;

    if (exception instanceof HttpException) {
      status = exception.getStatus();
      const exceptionResponse = exception.getResponse();
      
      if (typeof exceptionResponse === 'string') {
        message = exceptionResponse;
      } else if (typeof exceptionResponse === 'object') {
        message = (exceptionResponse as any).message || message;
        errors = (exceptionResponse as any).errors;
      }
    } else if (exception instanceof Error) {
      message = exception.message;
    }

    // Log error
    this.logger.error(
      `[${request.method}] ${request.url} - ${status} - ${message}`,
      exception instanceof Error ? exception.stack : '',
    );

    // Response
    response.status(status).json({
      success: false,
      statusCode: status,
      message,
      errors,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}

// Registrar globalmente
// main.ts
app.useGlobalFilters(new AllExceptionsFilter());
```

### Custom Exceptions

```typescript
// common/exceptions/business.exception.ts
import { HttpException, HttpStatus } from '@nestjs/common';

export class BusinessException extends HttpException {
  constructor(
    message: string,
    public readonly code: string,
    status: HttpStatus = HttpStatus.BAD_REQUEST,
  ) {
    super({ message, code }, status);
  }
}

// common/exceptions/entity-not-found.exception.ts
export class EntityNotFoundException extends HttpException {
  constructor(entity: string, id: string | number) {
    super(
      {
        message: `${entity} with ID ${id} not found`,
        code: 'ENTITY_NOT_FOUND',
        entity,
        id,
      },
      HttpStatus.NOT_FOUND,
    );
  }
}

// Uso
throw new EntityNotFoundException('User', userId);
throw new BusinessException('Insufficient balance', 'INSUFFICIENT_BALANCE');
```

---

## Middleware

```typescript
// common/middleware/logger.middleware.ts
import { Injectable, NestMiddleware, Logger } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  private logger = new Logger('HTTP');

  use(req: Request, res: Response, next: NextFunction) {
    const { method, originalUrl, ip } = req;
    const userAgent = req.get('user-agent') || '';
    const startTime = Date.now();

    res.on('finish', () => {
      const { statusCode } = res;
      const contentLength = res.get('content-length');
      const responseTime = Date.now() - startTime;

      this.logger.log(
        `${method} ${originalUrl} ${statusCode} ${contentLength} - ${responseTime}ms - ${userAgent} ${ip}`,
      );
    });

    next();
  }
}

// Functional middleware
export function correlationIdMiddleware(
  req: Request,
  res: Response,
  next: NextFunction,
) {
  const correlationId = req.headers['x-correlation-id'] as string || crypto.randomUUID();
  req['correlationId'] = correlationId;
  res.setHeader('x-correlation-id', correlationId);
  next();
}

// Aplicar middleware en módulo
@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware, correlationIdMiddleware)
      .exclude(
        { path: 'health', method: RequestMethod.GET },
        { path: 'metrics', method: RequestMethod.GET },
      )
      .forRoutes('*');
      
    // O para rutas específicas
    consumer
      .apply(AuthMiddleware)
      .forRoutes(
        { path: 'users', method: RequestMethod.ALL },
        UsersController,
      );
  }
}
```

---

## 🏷️ Tags

#programming #backend #nestjs #nodejs #typescript #api

---

## 📚 Ver También

- [[NestJS Advanced Guide|NestJS - Patrones Avanzados]]
- [[Next.js Complete Guide|Next.js - Guía Completa]]
