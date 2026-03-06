

### Creación de clases y objetos en c++
La cabecera de la clase es
```c++
#ifndef ACELEROMETRO_H
#define ACELEROMETRO_H
#include <Arduino.h>

class Acelerometro {
private:
    float x;  
    float y;  
    float z; 

public:
    Acelerometro(float x = 0.0, float y = 0.0, float z = 0.0);
    float getX() const;
    float getY() const;
    float getZ() const;

    void setX(float x);
    void setY(float y);
    void setZ(float z);
    
    float getMagnitud() const;

    String toJSON() const;
};

#endif 
```
La implementación de la clase es
```c++
#include <Arduino.h>
#include "Acelerometro.h"
#include <Arduino_JSON.h>
#include <cmath>  
  

// Constructor
Acelerometro::Acelerometro(float x, float y, float z) : x(x), y(y), z(z) {}

// Métodos para obtener los valores de los ejes
float Acelerometro::getX() const {
    return x;
}

float Acelerometro::getY() const {
    return y;
}

float Acelerometro::getZ() const {
    return z;
}

// Métodos para establecer los valores de los ejes
void Acelerometro::setX(float x) {
    this->x = x;
}

void Acelerometro::setY(float y) {
    this->y = y;
}

void Acelerometro::setZ(float z) {
    this->z = z;
}

// Método para obtener la magnitud de la aceleración
float Acelerometro::getMagnitud() const {
    return std::sqrt(x * x + y * y + z * z);
}

// Método para transformar a JSON
String Acelerometro::toJSON() const {
    JSONVar object;
    object["x"] = this->x;
    object["y"] = this->y;
    object["z"] = this->z;
    String jsonString = JSON.stringify(object);
    return jsonString;
}
```
Y para usar el objeto en cualquier contexto
```c++
#include <Acelerometro.h>
...
Acelerometro acelerometro(1.0, 2.0, 3.0);
```
