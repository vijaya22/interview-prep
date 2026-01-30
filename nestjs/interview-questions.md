# NestJS Interview Questions & Answers

---

## 1. What is NestJS?

NestJS is a progressive, opinionated Node.js framework for building scalable server-side applications. It is built with TypeScript and heavily inspired by Angular's architecture.

Key characteristics:
- **TypeScript-first** — full TypeScript support out of the box
- **Modular architecture** — organized into modules, controllers, and providers
- **Dependency injection** — built-in IoC (Inversion of Control) container
- **Platform-agnostic** — works with Express (default) or Fastify under the hood
- **Decorators-based** — uses decorators for metadata and routing
- **Opinionated** — enforces structure and best practices

---

## 2. What are the core building blocks of NestJS?

| Building Block | Purpose |
|---|---|
| **Module** | Organizes related components (controllers, providers) |
| **Controller** | Handles incoming HTTP requests and returns responses |
| **Provider/Service** | Contains business logic, injected via DI |
| **Middleware** | Runs before route handlers (like Express middleware) |
| **Guard** | Determines if a request should be handled (authorization) |
| **Interceptor** | Transforms request/response, adds extra logic |
| **Pipe** | Validates and transforms input data |
| **Filter** | Handles exceptions and error responses |
| **Decorator** | Attaches metadata to classes, methods, or parameters |

---

## 3. What is a Module in NestJS?

A module is a class decorated with `@Module()` that organizes the application into cohesive blocks.

```ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { AuthModule } from '../auth/auth.module';

@Module({
  imports: [AuthModule],           // other modules this module depends on
  controllers: [UsersController],  // controllers defined in this module
  providers: [UsersService],       // services/providers for DI
  exports: [UsersService],         // providers available to other modules
})
export class UsersModule {}
```

**Key concepts:**
- Every app has a root module (`AppModule`)
- Modules are singletons by default — the same instance is shared
- `imports` — bring in functionality from other modules
- `exports` — make providers available to importing modules
- Feature modules encapsulate related functionality

---

## 4. What is a Controller?

Controllers handle incoming HTTP requests and return responses to the client.

```ts
import {
  Controller, Get, Post, Put, Delete,
  Param, Query, Body, HttpCode, Header,
  Res, HttpStatus,
} from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users') // route prefix: /users
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll(@Query('page') page: number, @Query('limit') limit: number) {
    return this.usersService.findAll(page, limit);
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.usersService.update(id, updateUserDto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id') id: string) {
    return this.usersService.remove(id);
  }
}
```

---

## 5. What is a Provider/Service?

Providers are classes annotated with `@Injectable()` that contain business logic and can be injected into controllers or other providers.

```ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';
import { CreateUserDto } from './dto/create-user.dto';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly usersRepo: Repository<User>,
  ) {}

  async findAll(page = 1, limit = 10): Promise<User[]> {
    return this.usersRepo.find({
      skip: (page - 1) * limit,
      take: limit,
    });
  }

  async findOne(id: string): Promise<User> {
    const user = await this.usersRepo.findOne({ where: { id } });
    if (!user) {
      throw new NotFoundException(`User #${id} not found`);
    }
    return user;
  }

  async create(dto: CreateUserDto): Promise<User> {
    const user = this.usersRepo.create(dto);
    return this.usersRepo.save(user);
  }

  async update(id: string, dto: UpdateUserDto): Promise<User> {
    await this.findOne(id); // throws if not found
    await this.usersRepo.update(id, dto);
    return this.findOne(id);
  }

  async remove(id: string): Promise<void> {
    const result = await this.usersRepo.delete(id);
    if (result.affected === 0) {
      throw new NotFoundException(`User #${id} not found`);
    }
  }
}
```

---

## 6. What is Dependency Injection (DI) in NestJS?

NestJS has a built-in IoC container that manages class dependencies. You declare dependencies in the constructor, and NestJS resolves and injects them automatically.

```ts
// Standard injection — by type
@Injectable()
export class UsersService {
  constructor(private readonly dbService: DatabaseService) {}
}

// Custom token injection
@Injectable()
export class UsersService {
  constructor(@Inject('CONFIG') private config: AppConfig) {}
}

// Provider registration in module
@Module({
  providers: [
    // Standard (class provider)
    UsersService,

    // Value provider
    { provide: 'API_KEY', useValue: 'abc123' },

    // Factory provider
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async (configService: ConfigService) => {
        return createConnection(configService.get('DATABASE_URL'));
      },
      inject: [ConfigService],
    },

    // Existing provider (alias)
    { provide: 'AliasService', useExisting: UsersService },

    // Class provider
    {
      provide: UsersService,
      useClass: process.env.NODE_ENV === 'test'
        ? MockUsersService
        : UsersService,
    },
  ],
})
```

**Provider scopes:**
- `DEFAULT` — singleton (one instance shared across the app)
- `REQUEST` — new instance per request
- `TRANSIENT` — new instance per injection

```ts
@Injectable({ scope: Scope.REQUEST })
export class RequestScopedService {}
```

---

## 7. What are DTOs (Data Transfer Objects)?

DTOs define the shape of data sent over the network. Combined with `class-validator` and `class-transformer`, they validate and transform incoming data.

```ts
import {
  IsString, IsEmail, IsOptional, IsInt,
  MinLength, Min, Max, IsEnum,
} from 'class-validator';
import { Type, Transform } from 'class-transformer';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;

  @IsOptional()
  @IsInt()
  @Min(0)
  @Max(120)
  @Type(() => Number)
  age?: number;

  @IsOptional()
  @IsEnum(Role)
  role?: Role;

  @IsOptional()
  @Transform(({ value }) => value?.trim().toLowerCase())
  username?: string;
}

// Partial DTO for updates (all fields optional)
import { PartialType } from '@nestjs/mapped-types';

export class UpdateUserDto extends PartialType(CreateUserDto) {}

// Pick specific fields
import { PickType } from '@nestjs/mapped-types';

export class LoginDto extends PickType(CreateUserDto, ['email', 'password']) {}

// Omit specific fields
import { OmitType } from '@nestjs/mapped-types';

export class PublicUserDto extends OmitType(CreateUserDto, ['password']) {}
```

Enable global validation in `main.ts`:

```ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,       // strip unknown properties
  forbidNonWhitelisted: true, // throw on unknown properties
  transform: true,       // auto-transform to DTO types
}));
```

---

## 8. What are Pipes?

Pipes transform and validate input data before it reaches the route handler.

```ts
import {
  PipeTransform, Injectable, ArgumentMetadata,
  BadRequestException, ParseIntPipe, ParseUUIDPipe,
  DefaultValuePipe, ValidationPipe,
} from '@nestjs/common';

// Built-in pipes
@Get(':id')
findOne(@Param('id', ParseUUIDPipe) id: string) {
  return this.usersService.findOne(id);
}

@Get()
findAll(
  @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
  @Query('limit', new DefaultValuePipe(10), ParseIntPipe) limit: number,
) {
  return this.usersService.findAll(page, limit);
}

// Custom pipe
@Injectable()
export class ParseSortPipe implements PipeTransform {
  transform(value: string, metadata: ArgumentMetadata) {
    const allowed = ['name', 'email', 'createdAt'];
    if (!value) return 'createdAt';
    if (!allowed.includes(value)) {
      throw new BadRequestException(`Invalid sort field: ${value}`);
    }
    return value;
  }
}

@Get()
findAll(@Query('sort', ParseSortPipe) sort: string) {}

// Global validation pipe
app.useGlobalPipes(new ValidationPipe({ transform: true, whitelist: true }));
```

**Built-in pipes:** `ValidationPipe`, `ParseIntPipe`, `ParseFloatPipe`, `ParseBoolPipe`, `ParseArrayPipe`, `ParseUUIDPipe`, `ParseEnumPipe`, `DefaultValuePipe`, `ParseFilePipe`.

---

## 9. What are Guards?

Guards determine whether a request should be handled based on conditions like authentication or authorization.

```ts
import {
  CanActivate, ExecutionContext, Injectable,
  UnauthorizedException, SetMetadata,
} from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { JwtService } from '@nestjs/jwt';

// Auth guard
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);

    if (!token) throw new UnauthorizedException();

    try {
      const payload = await this.jwtService.verifyAsync(token);
      request.user = payload;
      return true;
    } catch {
      throw new UnauthorizedException();
    }
  }

  private extractToken(request: any): string | null {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : null;
  }
}

// Roles guard
export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(
      ROLES_KEY,
      [context.getHandler(), context.getClass()],
    );
    if (!requiredRoles) return true;

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some(role => user.roles?.includes(role));
  }
}

// Usage
@Controller('admin')
@UseGuards(AuthGuard, RolesGuard)
export class AdminController {
  @Get('dashboard')
  @Roles('admin')
  getDashboard() {
    return { message: 'Admin dashboard' };
  }
}
```

---

## 10. What are Interceptors?

Interceptors add extra logic before/after method execution. They can transform results, add logging, cache responses, or handle errors.

```ts
import {
  CallHandler, ExecutionContext, Injectable,
  NestInterceptor,
} from '@nestjs/common';
import { Observable, tap, map, timeout, catchError } from 'rxjs';

// Logging interceptor
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const now = Date.now();

    return next.handle().pipe(
      tap(() => {
        console.log(
          `${request.method} ${request.url} - ${Date.now() - now}ms`,
        );
      }),
    );
  }
}

// Transform response interceptor
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, any> {
  intercept(context: ExecutionContext, next: CallHandler<T>): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        statusCode: context.switchToHttp().getResponse().statusCode,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}

// Timeout interceptor
@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(timeout(5000));
  }
}

// Cache interceptor
@Injectable()
export class CacheInterceptor implements NestInterceptor {
  private cache = new Map<string, any>();

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const key = context.switchToHttp().getRequest().url;
    if (this.cache.has(key)) return of(this.cache.get(key));

    return next.handle().pipe(
      tap(data => this.cache.set(key, data)),
    );
  }
}

// Apply interceptors
@UseInterceptors(LoggingInterceptor, TransformInterceptor)
@Controller('users')
export class UsersController {}

// Global interceptor
app.useGlobalInterceptors(new LoggingInterceptor());
```

---

## 11. What are Exception Filters?

Exception filters handle errors thrown during request processing and format error responses.

```ts
import {
  ExceptionFilter, Catch, ArgumentsHost,
  HttpException, HttpStatus, NotFoundException,
} from '@nestjs/common';

// Catch all HTTP exceptions
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      message: exception.message,
      path: request.url,
      timestamp: new Date().toISOString(),
    });
  }
}

// Catch all exceptions (global fallback)
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    const message = exception instanceof HttpException
      ? exception.getResponse()
      : 'Internal server error';

    response.status(status).json({
      statusCode: status,
      message,
      path: request.url,
      timestamp: new Date().toISOString(),
    });
  }
}

// Usage
@UseFilters(HttpExceptionFilter)
@Controller('users')
export class UsersController {}

// Global filter
app.useGlobalFilters(new AllExceptionsFilter());
```

**Built-in HTTP exceptions:** `BadRequestException`, `UnauthorizedException`, `ForbiddenException`, `NotFoundException`, `ConflictException`, `GoneException`, `PayloadTooLargeException`, `UnprocessableEntityException`, `InternalServerErrorException`, `ServiceUnavailableException`.

---

## 12. What is Middleware in NestJS?

Middleware runs before route handlers. NestJS middleware is similar to Express middleware.

```ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
    next();
  }
}

// Functional middleware
export function corsMiddleware(req: Request, res: Response, next: NextFunction) {
  res.header('Access-Control-Allow-Origin', '*');
  next();
}

// Register in module
import { MiddlewareConsumer, NestModule, RequestMethod } from '@nestjs/common';

@Module({ /* ... */ })
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .exclude(
        { path: 'health', method: RequestMethod.GET },
      )
      .forRoutes(
        { path: 'users', method: RequestMethod.ALL },
        { path: 'posts', method: RequestMethod.GET },
      );

    // Apply to all routes
    consumer.apply(corsMiddleware).forRoutes('*');
  }
}
```

**Execution order:** Middleware → Guards → Interceptors (before) → Pipes → Route Handler → Interceptors (after) → Exception Filters.

---

## 13. What is the NestJS request lifecycle?

The request flows through these stages in order:

1. **Middleware** — runs first, can modify req/res or end the cycle
2. **Guards** — authorization check, can block the request
3. **Interceptors (before)** — pre-processing logic
4. **Pipes** — validate and transform input data
5. **Route Handler** — executes the controller method
6. **Interceptors (after)** — post-processing logic (transform response)
7. **Exception Filters** — catch and format errors (at any stage)

```
Request → Middleware → Guard → Interceptor (pre) → Pipe → Handler
                                                          ↓
Response ← Exception Filter ← Interceptor (post) ←───────┘
```

---

## 14. What are Custom Decorators?

NestJS allows creating custom decorators for extracting data, adding metadata, or composing existing decorators.

```ts
import { createParamDecorator, ExecutionContext, SetMetadata } from '@nestjs/common';

// Parameter decorator — extract current user
export const CurrentUser = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;
    return data ? user?.[data] : user;
  },
);

// Usage
@Get('profile')
getProfile(@CurrentUser() user: User) {
  return user;
}

@Get('name')
getName(@CurrentUser('name') name: string) {
  return { name };
}

// Metadata decorator
export const Public = () => SetMetadata('isPublic', true);

@Public()
@Get('health')
healthCheck() {
  return { status: 'ok' };
}

// Composed decorator
import { applyDecorators, UseGuards } from '@nestjs/common';

export function Auth(...roles: Role[]) {
  return applyDecorators(
    SetMetadata('roles', roles),
    UseGuards(AuthGuard, RolesGuard),
  );
}

@Auth(Role.Admin)
@Get('admin')
adminOnly() {}
```

---

## 15. What is the ConfigModule?

`@nestjs/config` manages environment variables and configuration.

```ts
// app.module.ts
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,              // available everywhere
      envFilePath: ['.env.local', '.env'],
      validationSchema: Joi.object({
        NODE_ENV: Joi.string().valid('development', 'production', 'test').required(),
        PORT: Joi.number().default(3000),
        DATABASE_URL: Joi.string().required(),
      }),
    }),
  ],
})
export class AppModule {}

// Using ConfigService
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getDatabaseUrl(): string {
    return this.configService.get<string>('DATABASE_URL');
  }
}

// Custom configuration files
// config/database.config.ts
export default registerAs('database', () => ({
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT, 10) || 5432,
  name: process.env.DB_NAME,
}));

// Import in module
ConfigModule.forRoot({
  load: [databaseConfig],
});

// Access namespaced config
const dbHost = this.configService.get<string>('database.host');
```

---

## 16. What is TypeORM integration in NestJS?

```ts
// app.module.ts
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        type: 'postgres',
        url: config.get('DATABASE_URL'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: config.get('NODE_ENV') !== 'production',
        logging: config.get('NODE_ENV') === 'development',
      }),
    }),
  ],
})
export class AppModule {}

// Entity
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn } from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;

  @Column({ select: false })
  password: string;

  @Column({ type: 'enum', enum: Role, default: Role.User })
  role: Role;

  @CreateDateColumn()
  createdAt: Date;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];
}

// Feature module
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}

// Service with repository
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepo: Repository<User>,
  ) {}

  async findAll(): Promise<User[]> {
    return this.usersRepo.find({ relations: ['posts'] });
  }
}
```

---

## 17. What is Prisma integration in NestJS?

```ts
// prisma.service.ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}

// prisma.module.ts
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}

// users.service.ts
@Injectable()
export class UsersService {
  constructor(private prisma: PrismaService) {}

  async findAll() {
    return this.prisma.user.findMany({
      include: { posts: true },
    });
  }

  async findOne(id: string) {
    const user = await this.prisma.user.findUnique({ where: { id } });
    if (!user) throw new NotFoundException();
    return user;
  }

  async create(dto: CreateUserDto) {
    return this.prisma.user.create({ data: dto });
  }

  async update(id: string, dto: UpdateUserDto) {
    return this.prisma.user.update({
      where: { id },
      data: dto,
    });
  }

  async remove(id: string) {
    return this.prisma.user.delete({ where: { id } });
  }
}
```

---

## 18. What is authentication in NestJS?

NestJS uses Passport.js via `@nestjs/passport` for authentication strategies.

```ts
// JWT Strategy
import { PassportStrategy } from '@nestjs/passport';
import { Strategy, ExtractJwt } from 'passport-jwt';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private configService: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get('JWT_SECRET'),
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, email: payload.email, role: payload.role };
  }
}

// Auth module
@Module({
  imports: [
    PassportModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        secret: config.get('JWT_SECRET'),
        signOptions: { expiresIn: '15m' },
      }),
    }),
    UsersModule,
  ],
  providers: [AuthService, JwtStrategy],
  controllers: [AuthController],
})
export class AuthModule {}

// Auth service
@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  async login(email: string, password: string) {
    const user = await this.usersService.findByEmail(email);
    if (!user || !(await bcrypt.compare(password, user.password))) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const payload = { sub: user.id, email: user.email, role: user.role };
    return {
      accessToken: this.jwtService.sign(payload),
      refreshToken: this.jwtService.sign(payload, { expiresIn: '7d' }),
    };
  }
}

// Auth controller
@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('login')
  login(@Body() dto: LoginDto) {
    return this.authService.login(dto.email, dto.password);
  }

  @UseGuards(AuthGuard('jwt'))
  @Get('profile')
  getProfile(@Request() req) {
    return req.user;
  }
}
```

---

## 19. What is the CQRS pattern in NestJS?

CQRS (Command Query Responsibility Segregation) separates read and write operations using `@nestjs/cqrs`.

```ts
// Command — write operation
export class CreateUserCommand {
  constructor(
    public readonly name: string,
    public readonly email: string,
  ) {}
}

@CommandHandler(CreateUserCommand)
export class CreateUserHandler implements ICommandHandler<CreateUserCommand> {
  constructor(private usersRepo: UsersRepository) {}

  async execute(command: CreateUserCommand): Promise<User> {
    const user = new User(command.name, command.email);
    return this.usersRepo.save(user);
  }
}

// Query — read operation
export class GetUserQuery {
  constructor(public readonly id: string) {}
}

@QueryHandler(GetUserQuery)
export class GetUserHandler implements IQueryHandler<GetUserQuery> {
  constructor(private usersRepo: UsersRepository) {}

  async execute(query: GetUserQuery): Promise<User> {
    return this.usersRepo.findOne(query.id);
  }
}

// Event — side effect
export class UserCreatedEvent {
  constructor(public readonly user: User) {}
}

@EventsHandler(UserCreatedEvent)
export class UserCreatedHandler implements IEventHandler<UserCreatedEvent> {
  handle(event: UserCreatedEvent) {
    console.log('Send welcome email to', event.user.email);
  }
}

// Controller usage
@Controller('users')
export class UsersController {
  constructor(
    private commandBus: CommandBus,
    private queryBus: QueryBus,
  ) {}

  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.commandBus.execute(new CreateUserCommand(dto.name, dto.email));
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.queryBus.execute(new GetUserQuery(id));
  }
}
```

---

## 20. What are WebSockets in NestJS?

NestJS provides a `@WebSocketGateway()` decorator for real-time communication.

```ts
import {
  WebSocketGateway, WebSocketServer,
  SubscribeMessage, MessageBody,
  ConnectedSocket, OnGatewayInit,
  OnGatewayConnection, OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({
  cors: { origin: '*' },
  namespace: '/chat',
})
export class ChatGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  handleConnection(client: Socket) {
    console.log(`Client connected: ${client.id}`);
  }

  handleDisconnect(client: Socket) {
    console.log(`Client disconnected: ${client.id}`);
  }

  @SubscribeMessage('message')
  handleMessage(
    @MessageBody() data: { room: string; message: string },
    @ConnectedSocket() client: Socket,
  ) {
    // Broadcast to room
    this.server.to(data.room).emit('message', {
      sender: client.id,
      message: data.message,
      timestamp: new Date(),
    });
  }

  @SubscribeMessage('joinRoom')
  handleJoinRoom(
    @MessageBody() room: string,
    @ConnectedSocket() client: Socket,
  ) {
    client.join(room);
    client.to(room).emit('userJoined', client.id);
  }
}
```

---

## 21. What are Microservices in NestJS?

NestJS has built-in support for microservice architectures with multiple transport layers.

```ts
// Microservice setup (TCP transport)
// main.ts
const app = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.TCP,
    options: { host: '0.0.0.0', port: 3001 },
  },
);
await app.listen();

// Message patterns
@Controller()
export class UsersController {
  @MessagePattern({ cmd: 'get_user' })
  getUser(@Payload() data: { id: string }) {
    return this.usersService.findOne(data.id);
  }

  @EventPattern('user_created')
  handleUserCreated(@Payload() data: CreateUserDto) {
    // Handle event (fire-and-forget)
    this.notificationService.sendWelcome(data.email);
  }
}

// Client (API Gateway)
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',
        transport: Transport.TCP,
        options: { host: 'users-service', port: 3001 },
      },
    ]),
  ],
})
export class GatewayModule {}

@Controller('users')
export class GatewayController {
  constructor(@Inject('USERS_SERVICE') private client: ClientProxy) {}

  @Get(':id')
  getUser(@Param('id') id: string) {
    return this.client.send({ cmd: 'get_user' }, { id });
  }

  @Post()
  createUser(@Body() dto: CreateUserDto) {
    this.client.emit('user_created', dto); // fire-and-forget
    return { message: 'User creation initiated' };
  }
}
```

**Supported transports:** TCP, Redis, NATS, MQTT, RabbitMQ, Kafka, gRPC.

---

## 22. What is the Scheduler module?

`@nestjs/schedule` enables cron jobs and intervals.

```ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression, Interval, Timeout } from '@nestjs/schedule';

@Injectable()
export class TasksService {
  // Cron job — runs at specific times
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  handleDailyCleanup() {
    console.log('Running daily cleanup...');
  }

  @Cron('0 */30 * * * *') // every 30 minutes
  handleDataSync() {
    console.log('Syncing data...');
  }

  // Interval — runs every N milliseconds
  @Interval(60000) // every minute
  handleHealthCheck() {
    console.log('Health check...');
  }

  // Timeout — runs once after N milliseconds
  @Timeout(5000) // 5 seconds after app start
  handleStartup() {
    console.log('App started, running initial setup...');
  }
}

// Register in module
@Module({
  imports: [ScheduleModule.forRoot()],
  providers: [TasksService],
})
export class AppModule {}
```

---

## 23. What is the Cache module?

NestJS provides a caching mechanism via `@nestjs/cache-manager`.

```ts
import { CacheModule, CacheInterceptor, CacheTTL, CacheKey } from '@nestjs/cache-manager';
import { redisStore } from 'cache-manager-redis-yet';

// Module setup
@Module({
  imports: [
    CacheModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        store: redisStore,
        host: config.get('REDIS_HOST'),
        port: config.get('REDIS_PORT'),
        ttl: 300, // 5 minutes default
      }),
    }),
  ],
})
export class AppModule {}

// Auto-cache responses with interceptor
@Controller('products')
@UseInterceptors(CacheInterceptor)
export class ProductsController {
  @Get()
  @CacheTTL(600) // 10 minutes
  @CacheKey('all-products')
  findAll() {
    return this.productsService.findAll();
  }
}

// Manual cache management
@Injectable()
export class ProductsService {
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}

  async findOne(id: string) {
    const cached = await this.cache.get(`product:${id}`);
    if (cached) return cached;

    const product = await this.productsRepo.findOne({ where: { id } });
    await this.cache.set(`product:${id}`, product, 3600);
    return product;
  }

  async update(id: string, dto: UpdateProductDto) {
    const product = await this.productsRepo.save({ id, ...dto });
    await this.cache.del(`product:${id}`);
    return product;
  }
}
```

---

## 24. What is the Event Emitter module?

`@nestjs/event-emitter` provides an event-driven architecture within a single application.

```ts
// Setup
@Module({
  imports: [EventEmitterModule.forRoot()],
})
export class AppModule {}

// Emit events
@Injectable()
export class OrdersService {
  constructor(private eventEmitter: EventEmitter2) {}

  async createOrder(dto: CreateOrderDto) {
    const order = await this.ordersRepo.save(dto);

    this.eventEmitter.emit('order.created', new OrderCreatedEvent(order));

    return order;
  }
}

// Listen to events
@Injectable()
export class NotificationsListener {
  @OnEvent('order.created')
  handleOrderCreated(event: OrderCreatedEvent) {
    // Send email notification
    this.emailService.send(event.order.userEmail, 'Order confirmed');
  }
}

@Injectable()
export class InventoryListener {
  @OnEvent('order.created')
  handleOrderCreated(event: OrderCreatedEvent) {
    // Update inventory
    this.inventoryService.decrementStock(event.order.items);
  }

  @OnEvent('order.created', { async: true })
  async handleAnalytics(event: OrderCreatedEvent) {
    // Async event handling
    await this.analyticsService.trackPurchase(event.order);
  }
}
```

---

## 25. What is Swagger/OpenAPI documentation in NestJS?

`@nestjs/swagger` auto-generates API documentation.

```ts
// main.ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

const config = new DocumentBuilder()
  .setTitle('My API')
  .setDescription('API documentation')
  .setVersion('1.0')
  .addBearerAuth()
  .addTag('users')
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api/docs', app, document);

// DTO with Swagger decorators
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';

export class CreateUserDto {
  @ApiProperty({ example: 'Alice', description: 'User name' })
  @IsString()
  name: string;

  @ApiProperty({ example: 'alice@example.com' })
  @IsEmail()
  email: string;

  @ApiPropertyOptional({ example: 25, minimum: 0, maximum: 120 })
  @IsOptional()
  @IsInt()
  age?: number;
}

// Controller with Swagger decorators
@ApiTags('users')
@ApiBearerAuth()
@Controller('users')
export class UsersController {
  @ApiOperation({ summary: 'Get all users' })
  @ApiQuery({ name: 'page', required: false, type: Number })
  @ApiResponse({ status: 200, description: 'List of users', type: [User] })
  @Get()
  findAll(@Query('page') page?: number) {}

  @ApiOperation({ summary: 'Create a user' })
  @ApiResponse({ status: 201, description: 'User created', type: User })
  @ApiResponse({ status: 400, description: 'Validation error' })
  @Post()
  create(@Body() dto: CreateUserDto) {}
}
```

---

## 26. What is file upload in NestJS?

NestJS uses multer under the hood for handling file uploads.

```ts
import {
  Controller, Post, UploadedFile, UploadedFiles,
  UseInterceptors, ParseFilePipe, MaxFileSizeValidator,
  FileTypeValidator,
} from '@nestjs/common';
import { FileInterceptor, FilesInterceptor } from '@nestjs/platform-express';
import { diskStorage } from 'multer';

@Controller('upload')
export class UploadController {
  // Single file
  @Post('avatar')
  @UseInterceptors(FileInterceptor('file', {
    storage: diskStorage({
      destination: './uploads',
      filename: (req, file, cb) => {
        const uniqueName = `${Date.now()}-${file.originalname}`;
        cb(null, uniqueName);
      },
    }),
  }))
  uploadAvatar(
    @UploadedFile(
      new ParseFilePipe({
        validators: [
          new MaxFileSizeValidator({ maxSize: 5 * 1024 * 1024 }), // 5MB
          new FileTypeValidator({ fileType: /(jpg|jpeg|png|webp)$/ }),
        ],
      }),
    )
    file: Express.Multer.File,
  ) {
    return { filename: file.filename, size: file.size };
  }

  // Multiple files
  @Post('gallery')
  @UseInterceptors(FilesInterceptor('files', 10))
  uploadGallery(@UploadedFiles() files: Express.Multer.File[]) {
    return files.map(f => ({ filename: f.filename, size: f.size }));
  }
}
```

---

## 27. What is the Health Check module?

`@nestjs/terminus` provides health check endpoints for monitoring.

```ts
import { TerminusModule } from '@nestjs/terminus';
import {
  HealthCheckService, HttpHealthIndicator,
  TypeOrmHealthIndicator, MemoryHealthIndicator,
  DiskHealthIndicator,
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private http: HttpHealthIndicator,
    private db: TypeOrmHealthIndicator,
    private memory: MemoryHealthIndicator,
    private disk: DiskHealthIndicator,
  ) {}

  @Get()
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
      () => this.http.pingCheck('api', 'https://api.example.com'),
      () => this.memory.checkHeap('memory_heap', 200 * 1024 * 1024), // 200MB
      () => this.disk.checkStorage('disk', { path: '/', thresholdPercent: 0.9 }),
    ]);
  }
}

// Response format
{
  "status": "ok",
  "info": {
    "database": { "status": "up" },
    "api": { "status": "up" },
    "memory_heap": { "status": "up" },
    "disk": { "status": "up" }
  }
}
```

---

## 28. What are Lifecycle Hooks in NestJS?

NestJS provides lifecycle hooks for modules, controllers, and providers.

```ts
import {
  OnModuleInit, OnModuleDestroy,
  OnApplicationBootstrap, OnApplicationShutdown,
  BeforeApplicationShutdown,
} from '@nestjs/common';

@Injectable()
export class AppService implements
  OnModuleInit,
  OnApplicationBootstrap,
  OnModuleDestroy,
  BeforeApplicationShutdown,
  OnApplicationShutdown
{
  // Called after the module's dependencies are resolved
  async onModuleInit() {
    console.log('Module initialized');
    await this.connectToDatabase();
  }

  // Called after all modules are initialized
  async onApplicationBootstrap() {
    console.log('Application bootstrapped');
    await this.seedDatabase();
  }

  // Called when the module is being destroyed
  async onModuleDestroy() {
    console.log('Module destroying');
  }

  // Called before shutdown signal handling
  async beforeApplicationShutdown(signal?: string) {
    console.log(`Before shutdown: ${signal}`);
    await this.finishPendingRequests();
  }

  // Called after connections are closed
  async onApplicationShutdown(signal?: string) {
    console.log(`Application shutdown: ${signal}`);
    await this.closeConnections();
  }
}

// Enable graceful shutdown
const app = await NestFactory.create(AppModule);
app.enableShutdownHooks();
```

---

## 29. What is the difference between `forRoot()` and `forFeature()`?

| Method | Purpose | Scope |
|---|---|---|
| `forRoot()` | Configure module globally (once in AppModule) | App-wide singleton |
| `forFeature()` | Register feature-specific resources | Feature module scoped |
| `forRootAsync()` | Async version of `forRoot()` (use factory for config) | App-wide singleton |

```ts
// AppModule — configure once
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      url: process.env.DATABASE_URL,
      entities: [User, Post],
    }),
    CacheModule.forRoot({ ttl: 300 }),
  ],
})
export class AppModule {}

// Feature module — register entities/features
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService],
})
export class UsersModule {}

// Async configuration with factory
@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        type: 'postgres',
        url: config.get('DATABASE_URL'),
      }),
    }),
  ],
})
export class AppModule {}
```

---

## 30. What is a Dynamic Module?

Dynamic modules are modules whose configuration is determined at import time.

```ts
import { DynamicModule, Module } from '@nestjs/common';

interface MailModuleOptions {
  apiKey: string;
  from: string;
}

@Module({})
export class MailModule {
  static forRoot(options: MailModuleOptions): DynamicModule {
    return {
      module: MailModule,
      global: true,
      providers: [
        { provide: 'MAIL_OPTIONS', useValue: options },
        MailService,
      ],
      exports: [MailService],
    };
  }

  static forRootAsync(options: {
    imports?: any[];
    inject?: any[];
    useFactory: (...args: any[]) => MailModuleOptions | Promise<MailModuleOptions>;
  }): DynamicModule {
    return {
      module: MailModule,
      global: true,
      imports: options.imports || [],
      providers: [
        {
          provide: 'MAIL_OPTIONS',
          useFactory: options.useFactory,
          inject: options.inject || [],
        },
        MailService,
      ],
      exports: [MailService],
    };
  }
}

// Usage
@Module({
  imports: [
    MailModule.forRoot({ apiKey: 'abc123', from: 'noreply@app.com' }),
    // or async
    MailModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        apiKey: config.get('MAIL_API_KEY'),
        from: config.get('MAIL_FROM'),
      }),
    }),
  ],
})
export class AppModule {}
```

---

## 31. What is GraphQL integration in NestJS?

NestJS supports GraphQL via `@nestjs/graphql` with code-first or schema-first approaches.

```ts
// Code-first approach
import { ObjectType, Field, Int, ID, Query, Mutation, Resolver, Args } from '@nestjs/graphql';

// Type definition
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  @Field()
  email: string;

  @Field(() => Int)
  age: number;

  @Field(() => [Post], { nullable: true })
  posts?: Post[];
}

// Input type
@InputType()
export class CreateUserInput {
  @Field()
  @IsString()
  name: string;

  @Field()
  @IsEmail()
  email: string;
}

// Resolver
@Resolver(() => User)
export class UsersResolver {
  constructor(private usersService: UsersService) {}

  @Query(() => [User], { name: 'users' })
  findAll() {
    return this.usersService.findAll();
  }

  @Query(() => User, { name: 'user' })
  findOne(@Args('id', { type: () => ID }) id: string) {
    return this.usersService.findOne(id);
  }

  @Mutation(() => User)
  createUser(@Args('input') input: CreateUserInput) {
    return this.usersService.create(input);
  }

  @ResolveField(() => [Post])
  posts(@Parent() user: User) {
    return this.postsService.findByUserId(user.id);
  }
}

// Module setup
@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: join(process.cwd(), 'schema.gql'),
      playground: true,
    }),
  ],
})
export class AppModule {}
```

---

## 32. What is Throttling/Rate Limiting in NestJS?

`@nestjs/throttler` provides rate limiting.

```ts
import { ThrottlerModule, ThrottlerGuard, Throttle, SkipThrottle } from '@nestjs/throttler';

// Module setup
@Module({
  imports: [
    ThrottlerModule.forRoot([
      {
        name: 'short',
        ttl: 1000,       // 1 second
        limit: 3,        // 3 requests per second
      },
      {
        name: 'long',
        ttl: 60000,      // 1 minute
        limit: 100,      // 100 requests per minute
      },
    ]),
  ],
  providers: [{ provide: APP_GUARD, useClass: ThrottlerGuard }],
})
export class AppModule {}

// Custom limits per route
@Controller('auth')
export class AuthController {
  @Throttle({ short: { limit: 1, ttl: 1000 } }) // stricter: 1 per second
  @Post('login')
  login() {}

  @SkipThrottle() // no rate limiting
  @Get('status')
  status() {}
}
```

---

## 33. What is Serialization in NestJS?

Serialization controls what data is sent in responses using `class-transformer`.

```ts
import { Exclude, Expose, Transform, Type } from 'class-transformer';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  @Column()
  @Exclude() // never sent in responses
  password: string;

  @Column()
  @Exclude()
  internalNote: string;

  @CreateDateColumn()
  @Transform(({ value }) => value.toISOString())
  createdAt: Date;

  @Expose() // computed property
  get displayName(): string {
    return `${this.name} (${this.email})`;
  }
}

// Enable globally
app.useGlobalInterceptors(new ClassSerializerInterceptor(app.get(Reflector)));

// Response groups
export class User {
  @Expose({ groups: ['admin'] })
  internalId: string;
}

@SerializeOptions({ groups: ['admin'] })
@Get('admin/users')
findAllForAdmin() {}
```

---

## 34. What are Queue/Jobs in NestJS?

`@nestjs/bull` or `@nestjs/bullmq` provides job queue functionality with Redis.

```ts
import { BullModule, InjectQueue, Process, Processor } from '@nestjs/bull';
import { Queue, Job } from 'bull';

// Module setup
@Module({
  imports: [
    BullModule.forRoot({
      redis: { host: 'localhost', port: 6379 },
    }),
    BullModule.registerQueue({ name: 'email' }),
  ],
  providers: [EmailProcessor],
})
export class EmailModule {}

// Producer — add jobs to queue
@Injectable()
export class NotificationService {
  constructor(@InjectQueue('email') private emailQueue: Queue) {}

  async sendWelcomeEmail(user: User) {
    await this.emailQueue.add('welcome', {
      to: user.email,
      name: user.name,
    }, {
      delay: 5000,          // delay 5 seconds
      attempts: 3,           // retry 3 times
      backoff: { type: 'exponential', delay: 1000 },
      removeOnComplete: true,
    });
  }

  async sendBulkEmails(users: User[]) {
    const jobs = users.map(user => ({
      name: 'welcome',
      data: { to: user.email, name: user.name },
    }));
    await this.emailQueue.addBulk(jobs);
  }
}

// Consumer — process jobs
@Processor('email')
export class EmailProcessor {
  @Process('welcome')
  async handleWelcome(job: Job<{ to: string; name: string }>) {
    const { to, name } = job.data;
    await this.mailer.send(to, `Welcome ${name}!`);
  }

  @OnQueueFailed()
  handleFailure(job: Job, error: Error) {
    console.error(`Job ${job.id} failed:`, error.message);
  }

  @OnQueueCompleted()
  handleComplete(job: Job) {
    console.log(`Job ${job.id} completed`);
  }
}
```

---

## 35. What is versioning in NestJS?

NestJS supports API versioning out of the box.

```ts
// Enable versioning in main.ts
app.enableVersioning({
  type: VersioningType.URI,       // /v1/users, /v2/users
  // type: VersioningType.HEADER,  // X-API-Version: 1
  // type: VersioningType.MEDIA_TYPE, // Accept: application/json;v=1
  defaultVersion: '1',
});

// Controller-level versioning
@Controller({ path: 'users', version: '1' })
export class UsersV1Controller {
  @Get()
  findAll() {
    return 'V1: basic user list';
  }
}

@Controller({ path: 'users', version: '2' })
export class UsersV2Controller {
  @Get()
  findAll() {
    return 'V2: enhanced user list with pagination';
  }
}

// Route-level versioning
@Controller('users')
export class UsersController {
  @Version('1')
  @Get()
  findAllV1() {
    return 'V1 response';
  }

  @Version('2')
  @Get()
  findAllV2() {
    return 'V2 response';
  }

  @Version(VERSION_NEUTRAL) // available in all versions
  @Get('health')
  health() {
    return { status: 'ok' };
  }
}
```

---

## 36. What is testing in NestJS?

NestJS has built-in testing utilities via `@nestjs/testing`.

```ts
import { Test, TestingModule } from '@nestjs/testing';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';
import { getRepositoryToken } from '@nestjs/typeorm';
import { User } from './entities/user.entity';

describe('UsersController', () => {
  let controller: UsersController;
  let service: UsersService;

  const mockUsersService = {
    findAll: jest.fn().mockResolvedValue([{ id: '1', name: 'Alice' }]),
    findOne: jest.fn().mockResolvedValue({ id: '1', name: 'Alice' }),
    create: jest.fn().mockResolvedValue({ id: '1', name: 'Alice' }),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        { provide: UsersService, useValue: mockUsersService },
      ],
    }).compile();

    controller = module.get<UsersController>(UsersController);
    service = module.get<UsersService>(UsersService);
  });

  it('should return all users', async () => {
    const result = await controller.findAll();
    expect(result).toHaveLength(1);
    expect(service.findAll).toHaveBeenCalled();
  });
});

// E2E testing
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';

describe('Users (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = module.createNestApplication();
    app.useGlobalPipes(new ValidationPipe());
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  it('GET /users should return 200', () => {
    return request(app.getHttpServer())
      .get('/users')
      .expect(200)
      .expect((res) => {
        expect(Array.isArray(res.body)).toBe(true);
      });
  });

  it('POST /users with invalid data should return 400', () => {
    return request(app.getHttpServer())
      .post('/users')
      .send({ name: '' })
      .expect(400);
  });
});
```

---

## 37. What is the Logger in NestJS?

NestJS provides a built-in logger and supports custom loggers.

```ts
import { Logger, Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
  private readonly logger = new Logger(UsersService.name);

  async findOne(id: string) {
    this.logger.log(`Finding user ${id}`);
    this.logger.warn(`User ${id} has expired token`);
    this.logger.error(`User ${id} not found`, 'stack trace here');
    this.logger.debug(`Query params: ${JSON.stringify(params)}`);
    this.logger.verbose(`Detailed info for debugging`);
  }
}

// Custom logger implementation
import { LoggerService } from '@nestjs/common';

export class CustomLogger implements LoggerService {
  log(message: string, context?: string) {
    // Send to logging service (e.g., Winston, Pino)
  }
  error(message: string, trace?: string, context?: string) {}
  warn(message: string, context?: string) {}
  debug(message: string, context?: string) {}
  verbose(message: string, context?: string) {}
}

// Use custom logger
const app = await NestFactory.create(AppModule, {
  logger: new CustomLogger(),
});

// Or with Pino
import { Logger as PinoLogger } from 'nestjs-pino';

const app = await NestFactory.create(AppModule, { bufferLogs: true });
app.useLogger(app.get(PinoLogger));
```

---

## 38. What is the difference between Express and Fastify adapters?

NestJS is platform-agnostic and supports both Express and Fastify as underlying HTTP frameworks.

| Feature | Express | Fastify |
|---|---|---|
| Performance | Good | Better (up to 2x faster) |
| Ecosystem | Massive (most middleware) | Growing |
| Default in NestJS | Yes | No |
| TypeScript | Types via `@types/express` | Built-in TypeScript |
| Schema validation | External (Joi, class-validator) | Built-in (JSON Schema) |

```ts
// Default Express adapter
const app = await NestFactory.create(AppModule);

// Fastify adapter
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';

const app = await NestFactory.create<NestFastifyApplication>(
  AppModule,
  new FastifyAdapter(),
);
await app.listen(3000, '0.0.0.0'); // Fastify needs '0.0.0.0' for Docker
```

---

## 39. What are Circular Dependencies and how to resolve them?

Circular dependencies occur when two modules or providers depend on each other.

```ts
// Problem: A depends on B, and B depends on A

// Solution 1: forwardRef()
@Module({
  imports: [forwardRef(() => ModuleB)],
})
export class ModuleA {}

@Module({
  imports: [forwardRef(() => ModuleA)],
})
export class ModuleB {}

// Provider-level
@Injectable()
export class ServiceA {
  constructor(
    @Inject(forwardRef(() => ServiceB))
    private serviceB: ServiceB,
  ) {}
}

// Solution 2: Restructure — extract shared logic into a third module
@Module({
  providers: [SharedService],
  exports: [SharedService],
})
export class SharedModule {}

// Solution 3: Use events instead of direct dependencies
// ModuleA emits event → ModuleB listens
```

Best practice: circular dependencies usually indicate a design problem. Restructure modules to avoid them.

---

## 40. What is the Execution Context?

`ExecutionContext` extends `ArgumentsHost` and provides metadata about the current execution context. It is used in guards, interceptors, and filters.

```ts
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get handler and class metadata
    const isPublic = this.reflector.getAllAndOverride<boolean>('isPublic', [
      context.getHandler(),  // method decorator metadata
      context.getClass(),    // class decorator metadata
    ]);

    if (isPublic) return true;

    // Access request based on context type
    const type = context.getType(); // 'http' | 'ws' | 'rpc'

    if (type === 'http') {
      const request = context.switchToHttp().getRequest();
      const response = context.switchToHttp().getResponse();
      return this.validateHttpRequest(request);
    }

    if (type === 'ws') {
      const client = context.switchToWs().getClient();
      const data = context.switchToWs().getData();
      return this.validateWsConnection(client);
    }

    if (type === 'rpc') {
      const data = context.switchToRpc().getData();
      const ctx = context.switchToRpc().getContext();
      return this.validateRpcRequest(data);
    }

    return false;
  }
}
```

---

## 41. What is Lazy Loading of Modules?

Lazy loading defers module initialization until it's needed, improving startup time.

```ts
import { LazyModuleLoader } from '@nestjs/core';

@Injectable()
export class AppService {
  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  async generateReport() {
    // Module is loaded only when this method is called
    const { ReportModule } = await import('./report/report.module');
    const moduleRef = await this.lazyModuleLoader.load(() => ReportModule);

    const reportService = moduleRef.get(ReportService);
    return reportService.generate();
  }
}
```

Use cases:
- Serverless functions (reduce cold start time)
- Heavy modules used infrequently (report generation, PDF creation)
- Optional features based on configuration

---

## 42. What is the Reflector class?

`Reflector` is a helper for accessing metadata set by decorators. It is primarily used in guards and interceptors.

```ts
import { Reflector } from '@nestjs/core';
import { SetMetadata } from '@nestjs/common';

// Custom metadata decorator
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
export const Public = () => SetMetadata('isPublic', true);

// Read metadata in guard
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get from handler only
    const roles = this.reflector.get<string[]>('roles', context.getHandler());

    // Get from handler, fallback to class
    const roles = this.reflector.getAllAndOverride<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);

    // Merge from handler and class
    const roles = this.reflector.getAllAndMerge<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!roles) return true;

    const user = context.switchToHttp().getRequest().user;
    return roles.some(role => user.roles?.includes(role));
  }
}
```

---

## 43. What is Server-Sent Events (SSE) in NestJS?

SSE enables server-to-client streaming over HTTP.

```ts
import { Controller, Sse, MessageEvent } from '@nestjs/common';
import { Observable, interval, map } from 'rxjs';

@Controller('events')
export class EventsController {
  @Sse('stream')
  stream(): Observable<MessageEvent> {
    return interval(1000).pipe(
      map((num) => ({
        data: { count: num, timestamp: new Date().toISOString() },
        type: 'counter',
        id: String(num),
      })),
    );
  }

  @Sse('notifications')
  notifications(): Observable<MessageEvent> {
    return this.notificationsService.getStream().pipe(
      map((notification) => ({
        data: notification,
        type: 'notification',
      })),
    );
  }
}
```

Client-side:
```js
const events = new EventSource('/events/stream');
events.addEventListener('counter', (e) => {
  console.log(JSON.parse(e.data));
});
```

---

## 44. What is the difference between Guards, Interceptors, Pipes, and Middleware?

| Feature | Middleware | Guard | Interceptor | Pipe |
|---|---|---|---|---|
| Purpose | Generic request processing | Authorization/access control | Transform request/response | Validate/transform input |
| Access to | `req`, `res`, `next` | `ExecutionContext` | `ExecutionContext`, `CallHandler` | Value + metadata |
| Can stop request | Yes (`next()` not called) | Yes (return `false`) | Yes (don't call `next.handle()`) | Yes (throw exception) |
| Has DI | Yes | Yes | Yes | Yes |
| Runs | Before everything | After middleware | Around handler | Before handler |
| Response access | Yes | No | Yes (via RxJS) | No |

**When to use what:**
- **Middleware** — logging, CORS, body parsing, request-level transformations
- **Guard** — authentication, role-based access, feature flags
- **Interceptor** — response mapping, caching, logging with timing, error handling
- **Pipe** — input validation (DTOs), type transformation

---

## 45. What is Streaming in NestJS?

```ts
import { Controller, Get, Res, StreamableFile } from '@nestjs/common';
import { createReadStream } from 'fs';
import { join } from 'path';

@Controller('files')
export class FilesController {
  // StreamableFile (platform-agnostic, recommended)
  @Get('download')
  getFile(): StreamableFile {
    const file = createReadStream(join(process.cwd(), 'package.json'));
    return new StreamableFile(file, {
      type: 'application/json',
      disposition: 'attachment; filename="package.json"',
    });
  }

  // Generate CSV stream
  @Get('report')
  async getReport(): Promise<StreamableFile> {
    const data = await this.reportService.generateCSV();
    const stream = Readable.from(data);
    return new StreamableFile(stream, {
      type: 'text/csv',
      disposition: 'attachment; filename="report.csv"',
    });
  }
}
```

---

## 46. What is the `@nestjs/config` namespace feature?

Namespaced configuration organizes settings into logical groups.

```ts
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT, 10) || 5432,
  name: process.env.DB_NAME || 'mydb',
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
}));

// config/jwt.config.ts
export default registerAs('jwt', () => ({
  secret: process.env.JWT_SECRET,
  expiresIn: process.env.JWT_EXPIRES_IN || '15m',
}));

// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig, jwtConfig],
    }),
  ],
})
export class AppModule {}

// Inject namespaced config
@Injectable()
export class DatabaseService {
  constructor(
    @Inject(databaseConfig.KEY)
    private dbConfig: ConfigType<typeof databaseConfig>,
  ) {
    console.log(this.dbConfig.host); // fully typed
    console.log(this.dbConfig.port);
  }
}
```

---

## 47. What are Custom Providers?

NestJS supports several ways to provide dependencies beyond simple class injection.

```ts
@Module({
  providers: [
    // 1. Standard class provider
    UsersService,

    // 2. Value provider — inject a static value
    { provide: 'APP_NAME', useValue: 'My Application' },

    // 3. Class provider — conditionally choose implementation
    {
      provide: PaymentService,
      useClass: process.env.NODE_ENV === 'test'
        ? MockPaymentService
        : StripePaymentService,
    },

    // 4. Factory provider — create instance with custom logic
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async (config: ConfigService) => {
        const connection = await createConnection({
          url: config.get('DATABASE_URL'),
        });
        return connection;
      },
      inject: [ConfigService],
    },

    // 5. Existing provider — alias
    { provide: 'LOGGER', useExisting: LoggerService },

    // 6. Async factory with multiple dependencies
    {
      provide: 'CACHE_CLIENT',
      useFactory: async (config: ConfigService, logger: LoggerService) => {
        const client = new Redis(config.get('REDIS_URL'));
        logger.log('Redis connected');
        return client;
      },
      inject: [ConfigService, LoggerService],
    },
  ],
})
export class AppModule {}
```

---

## 48. What are common NestJS project structures?

**Feature-based structure (recommended):**

```
src/
├── app.module.ts
├── main.ts
├── common/
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   ├── pipes/
│   └── middleware/
├── config/
│   ├── database.config.ts
│   └── jwt.config.ts
├── users/
│   ├── users.module.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   ├── dto/
│   │   ├── create-user.dto.ts
│   │   └── update-user.dto.ts
│   ├── entities/
│   │   └── user.entity.ts
│   └── users.controller.spec.ts
├── auth/
│   ├── auth.module.ts
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── strategies/
│   │   └── jwt.strategy.ts
│   └── guards/
│       └── auth.guard.ts
└── posts/
    ├── posts.module.ts
    ├── posts.controller.ts
    └── posts.service.ts
```

Each feature module is self-contained with its own controller, service, DTOs, entities, and tests.

---

## 49. What are common NestJS anti-patterns?

1. **Fat controllers** — put business logic in services, not controllers
2. **Not using DTOs** — leads to unvalidated input and missing documentation
3. **Circular dependencies** — restructure modules or use `forwardRef()` as last resort
4. **Not using global pipes** — validate input on every route with `ValidationPipe`
5. **Hardcoding configuration** — use `ConfigModule` and environment variables
6. **Not using `@Injectable()` scope properly** — avoid `REQUEST` scope unless truly needed (it ripples through the dependency graph)
7. **Ignoring error handling** — use exception filters for consistent error responses
8. **Not testing** — NestJS has excellent testing utilities, use them
9. **Over-engineering** — not every app needs CQRS, microservices, or event sourcing
10. **Not enabling shutdown hooks** — call `app.enableShutdownHooks()` for graceful shutdown

---

## 50. What NestJS questions should you ask about a company's setup?

When interviewing, consider asking:
- Which database and ORM do you use (TypeORM, Prisma, Drizzle)?
- Do you use monorepo or multi-repo for microservices?
- What transport layer for microservices (TCP, Redis, RabbitMQ, Kafka)?
- How do you handle authentication (Passport, custom guards)?
- What is your testing strategy (unit, e2e, coverage requirements)?
- Do you use GraphQL or REST (or both)?
- How do you handle background jobs (BullMQ, custom queues)?
- What is your deployment pipeline (Docker, Kubernetes, serverless)?
- Do you use NestJS CLI for generating resources?
- Which version of NestJS are you on and do you keep up with updates?
