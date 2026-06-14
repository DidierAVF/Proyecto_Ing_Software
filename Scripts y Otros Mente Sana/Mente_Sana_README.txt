========================================================================
SISTEMA DE GESTIÓN Y FACTURACIÓN - CENTRO PSICOLÓGICO MENTE SANA
MenteSana Web | Desarrollo por NovaBit
========================================================================

Este archivo constituye la guía oficial de configuración, prerrequisitos e inicialización del entorno técnico del sistema de manera unificada bajo una arquitectura de monorrepósito.

1. DESCRIPCIÓN GENERAL DEL SISTEMA
------------------------------------------------------------------------
El sistema es una plataforma web multiusuario concebida para automatizar las operaciones críticas del Centro Psicológico Mente Sana ubicado en Cúcuta, Norte de Santander. El software restringe su acceso a tres perfiles operativos estrictos (Administrador, Especialista/Psicólogo y Recepción) y cubre las necesidades de:
- Control de agenda con bloqueo transaccional ante cruces de horarios.
- Expediente de historias clínicas digitales con bitácora inmodificable de auditoría.
- Facturación interna controlada mediante numeración consecutiva.
- Control de flujos de caja diaria para recaudos en efectivo, Nequi y Bancolombia.

2. STACK TECNOLÓGICO
------------------------------------------------------------------------
- Frontend: Angular 17 + TypeScript (Arquitectura basada en componentes y módulos).
- Backend: NestJS + TypeScript (Framework REST sobre Node.js).
- Base de Datos: PostgreSQL 15 (Motor relacional estructurado).
- Autenticación: JSON Web Tokens (JWT) firmados mediante algoritmo HS256.

3. ESTRUCTURA DEL REPOSITORIO (MONORREPÓSITO)
------------------------------------------------------------------------
mentesana-monorepo/
├── backend/                  # Servidor de Aplicaciones API REST (NestJS)
│   ├── src/
│   │   ├── auth/             # Módulo de seguridad, cifrado y tokens JWT
│   │   ├── agenda/           # Control del calendario y validación de horarios
│   │   ├── clinico/          # Historias clínicas y registros de auditoría
│   │   └── facturacion/      # Cobros, consecutivos y flujo de caja diaria
│   └── tsconfig.json         # Configuración del compilador de TypeScript
└── frontend/                 # Interfaz de Usuario Web (Angular 17)
    ├── src/
    │   ├── app/
    │   │   ├── core/         # Interceptores, guardianes de rol y servicios API
    │   │   ├── shared/       # Componentes y directivas reutilizables
    │   │   └── modulos/      # Vistas independientes por rol
    │   └── assets/           # Hojas de estilo unificadas y recursos estáticos
    └── angular.json          # Especificación de construcción de la SPA

4. PRERREQUISITOS DEL ENTORNO LOCAL
------------------------------------------------------------------------
Antes de iniciar la instalación, asegúrese de tener desplegados los siguientes componentes de software en el sistema anfitrión:
1. Node.js (Versión LTS v18 o superior).
2. NPM (Administrador de paquetes nativo de Node.js).
3. PostgreSQL 15 (Instancia activa del motor de base de datos).
4. pgAdmin 4 (Herramienta de administración para bases de datos PostgreSQL).

5. CONFIGURACIÓN DE VARIABLES DE ENTORNO
------------------------------------------------------------------------
Debe crear un archivo de configuración secreto denominado `.env` dentro de la ruta raíz del directorio `/backend`. Las variables requeridas corresponden a los siguientes parámetros lógicos:

# Conexión a la base de datos PostgreSQL
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USER=postgres
DATABASE_PASSWORD=Indique_Su_Contrasena_De_Postgres
DATABASE_NAME=mentesana_db

# Firma de seguridad para tokens de sesión
JWT_SECRET=NovaBit_MenteSana_HS256_SecretKey_2026
JWT_EXPIRATION_TIME=28800s

# Control perimetral de acceso
MAX_LOGIN_ATTEMPTS=3
LOCKOUT_DURATION_MINUTES=15

6. PASOS PARA LA INICIALIZACIÓN Y DESPLIEGUE
------------------------------------------------------------------------

FASE 1: PREPARACIÓN DE LA BASE DE DATOS
1. Inicie la herramienta pgAdmin 4 y conéctese al servidor de base de datos local.
2. Genere una base de datos nueva y asígnele el nombre exacto de: mentesana_db
3. Abra una ventana de consultas (Query Tool) sobre la base de datos creada.
4. Cargue y ejecute en su totalidad el archivo denominado "Script para Base de Datos Mente Sana.txt" ubicado en el repositorio. Este script activará las restricciones de seguridad, creará las tablas jerárquicas e insertará los registros maestros iniciales de roles, especialidades y métodos de pago.

FASE 2: COMPILACIÓN Y ARRANQUE DEL BACKEND (NestJS)
Abra una terminal del sistema y ejecute la siguiente secuencia de comandos:
> cd backend
> npm install
> npm run start:dev

La API REST se compilará de manera óptima y quedará escuchando peticiones en la dirección: http://localhost:3000

FASE 3: COMPILACIÓN Y ARRANQUE DEL FRONTEND (Angular 17)
Abra una segunda terminal del sistema de forma simultánea y ejecute:
> cd frontend
> npm install
> npm run start

La interfaz de usuario del sistema se inicializará correctamente y estará lista para operar a través de cualquier navegador web local en la ruta: http://localhost:4200

7. MATRIZ DE CREDENCIALES PRECONFIGURADAS PARA PRUEBAS
------------------------------------------------------------------------
El script de inicialización incluye tres usuarios predeterminados asociados a los distintos perfiles de la plataforma para validar el control de accesos:

- Perfil: ADMINISTRADOR
  Correo: valentina.rios@mentesana.com
  Contraseña: AdminMenteSana2026*
  Alcance: Cierre de caja, reportes financieros, edición de usuarios y auditoría clínica.

- Perfil: ESPECIALISTA/PSICÓLOGO
  Correo: psicologo.prueba@mentesana.com
  Contraseña: PsiMenteSana2026*
  Alcance: Agenda personal, edición de evoluciones y visualización de historias clínicas.

- Perfil: RECEPCIÓN
  Correo: recepcion.prueba@mentesana.com
  Contraseña: RecMenteSana2026*
  Alcance: Apertura de caja diaria, agendamiento de citas, cobros y facturación.

8. INFRAESTRUCTURA EN LA NUBE (TERRAFORM - AWS)
------------------------------------------------------------------------
La configuración de infraestructura como código define los recursos AWS necesarios
para el despliegue en producción del sistema. El archivo principal se denomina
"main.tf" y debe ubicarse en la raíz del monorrepósito.

# main.tf - Configuración de Infraestructura para Centro Psicológico Mente Sana
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
provider "aws" {
  region = var.region_despliegue
}
# Red Virtual Principal
resource "aws_vpc" "vpc_mente_sana" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  tags = {
    Name        = "vpc-mente-sana-prod"
    Environment = "Production"
    Project     = "Mente Sana"
    Developer   = "NovaBit"
  }
}
# Subred Privada para la Base de Datos PostgreSQL
resource "aws_subnet" "subnet_db_privada" {
  vpc_id            = aws_vpc.vpc_mente_sana.id
  cidr_block        = "10.0.4.0/24"
  availability_zone = "${var.region_despliegue}a"
  tags = {
    Name = "subnet-mente-sana-db-private"
  }
}
# Grupo de Seguridad para la Base de Datos
resource "aws_security_group" "sg_postgresql" {
  name        = "sg_mente_sana_postgres"
  description = "Control de acceso al motor de base de datos"
  vpc_id      = aws_vpc.vpc_mente_sana.id
  # Permitir trafico entrante solo desde la subred de la API NestJS
  ingress {
    description = "Conexiones PostgreSQL desde la API de NestJS"
    from_port   = 5432
    to_port     = 5432
    protocol    = "tcp"
    cidr_blocks = ["10.0.3.0/24"]
  }
  # Bloquear todo el trafico saliente por defecto de la base de datos
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "sg-mente-sana-postgres"
  }
}
# Instancia de Base de Datos PostgreSQL 15
resource "aws_db_instance" "postgres_mente_sana" {
  identifier             = "postgres-mentesana-db"
  allocated_storage      = 50
  max_allocated_storage  = 200
  engine                 = "postgres"
  engine_version         = "15.4"
  instance_class         = "db-t3-large"
  db_name                = "mentesana_prod"
  username               = var.db_usuario
  password               = var.db_contrasena
  db_subnet_group_name   = aws_db_subnet_group.db_group.name
  vpc_security_group_ids = [aws_security_group_sg_postgresql.id]
  skip_final_snapshot    = true
  tags = {
    Name = "database-mente-sana"
  }
}

========================================================================
PROPIEDAD DE NOVABIT - DOCUMENTACIÓN TÉCNICA DE CONSTRUCCIÓN DE SOFTWARE
========================================================================