# Modelo de Amenazas (Threat Model)

## 1. Arquitectura del Sistema

```mermaid
graph TD
    A[Usuario / Cliente] -->|Peticiones HTTPS| B[API Gateway / Backend]
    B -->|Consultas SQL| C[(Base de Datos)]
    
    %% INTENTO DE SABOTAJE / VULNERABILIDAD INTENCIONAL
    A -->|Acceso Directo Sin Autenticación| C
Usuario - de Datos (Acceso directo sin API) 
Usuario - de Datos (Acceso directo sin API) 
Prueba de bloqueo 
