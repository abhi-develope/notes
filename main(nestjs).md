# NestJS Project – `main.ts` Setup

This project uses a custom `main.ts` bootstrap file with the following features:

- ✅ Global Validation with `ValidationPipe`
- 🚀 Swagger documentation via `@nestjs/swagger`
- 🌐 CORS enabled for all origins
- ⚙️ Environment-based config via `@nestjs/config`
- 📌 API routes prefixed with `/api`

## `main.ts`

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Set global prefix for all routes
  app.setGlobalPrefix('api');

  // Enable CORS for all origins
  app.enableCors();

  // Use validation globally
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      transform: true,
    }),
  );

  // Swagger setup
  const config = new DocumentBuilder()
    .setTitle('My API')
    .setDescription('API documentation')
    .setVersion('1.0')
    .addBearerAuth()
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api/swagger', app, document);

  // Get PORT from environment
  const configService = app.get(ConfigService);
  const PORT = configService.get('PORT') || 3000;

  await app.listen(PORT);
  console.log(`🚀 Server is running on http://localhost:${PORT}`);
}

bootstrap();
