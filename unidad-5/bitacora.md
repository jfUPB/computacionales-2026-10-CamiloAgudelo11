# Unidad 5



## Bitácora de aplicación 


### Fase 1


#### ofApp.h


```

#pragma once
#include "ofMain.h"
#include <vector>

class Particle {
public:
	virtual ~Particle() { }
	virtual void update(float dt) = 0;
	virtual void draw() = 0;
	virtual bool isDead() const = 0;
	virtual bool shouldExplode() const { return false; }
	virtual glm::vec2 getPosition() const { return glm::vec2(0, 0); }
	virtual ofColor getColor() const { return ofColor(255); }
};

class RisingParticle : public Particle {
protected:
	glm::vec2 position;
	glm::vec2 velocity;
	ofColor color;
	float lifetime;
	float age;
	bool exploded;

public:
	RisingParticle(const glm::vec2 & pos, const glm::vec2 & vel,
		const ofColor & col, float life)
		: position(pos)
		, velocity(vel)
		, color(col)
		, lifetime(life)
		, age(0)
		, exploded(false) { }

	void update(float dt) override {
		position += velocity * dt;
		age += dt;
		velocity.y += 9.8f * dt * 8;
		float explosionThreshold = ofGetHeight() * 0.15f + ofRandom(-30, 30);
		if (position.y <= explosionThreshold || age >= lifetime) {
			exploded = true;
		}
	}

	void draw() override {
		ofSetColor(color);
		ofDrawCircle(position, 10);
	}

	bool isDead() const override { return exploded; }
	bool shouldExplode() const override { return exploded; }
	glm::vec2 getPosition() const override { return position; }
	ofColor getColor() const override { return color; }
};

class ZigZagParticle : public Particle {
	glm::vec2 position;
	glm::vec2 velocity;
	ofColor color;
	float age;
	float lifetime;

public:
	ZigZagParticle(const glm::vec2 & pos)
		: position(pos)
		, velocity(glm::vec2(0, -200))
		, age(0)
		, lifetime(2.0f) {
		color.setHsb(ofRandom(255), 200, 255);
	}

	void update(float dt) override {
		age += dt;
		position.y += velocity.y * dt;
		position.x += sin(age * 10) * 5;
	}

	void draw() override {
		ofSetColor(color);
		ofDrawCircle(position, 6);
	}

	bool isDead() const override { return age >= lifetime; }
};

class SpiralParticle : public Particle {
	glm::vec2 center;
	float angle;
	float radius;
	float age;
	float lifetime;
	ofColor color;

public:
	SpiralParticle(const glm::vec2 & c)
		: center(c)
		, angle(0)
		, radius(0)
		, age(0)
		, lifetime(2.5f) {
		color.setHsb(ofRandom(255), 255, 255);
	}

	void update(float dt) override {
		age += dt;
		angle += 5 * dt;
		radius += 20 * dt;
	}

	void draw() override {
		glm::vec2 pos = center + glm::vec2(cos(angle), sin(angle)) * radius;
		ofSetColor(color);
		ofDrawCircle(pos, 5);
	}

	bool isDead() const override { return age >= lifetime; }
};

class ExplosionParticle : public Particle {
protected:
	glm::vec2 position;
	glm::vec2 velocity;
	ofColor color;
	float age;
	float lifetime;
	float size;

public:
	ExplosionParticle(const glm::vec2 & pos, const glm::vec2 & vel,
		const ofColor & col, float life, float sz)
		: position(pos)
		, velocity(vel)
		, color(col)
		, age(0)
		, lifetime(life)
		, size(sz) { }

	void update(float dt) override {
		position += velocity * dt;
		age += dt;
		float alpha = ofMap(age, 0, lifetime, 255, 0, true);
		color.a = alpha;
	}

	bool isDead() const override { return age >= lifetime; }
};

class CircularExplosion : public ExplosionParticle {
public:
	CircularExplosion(const glm::vec2 & pos, const ofColor & col)
		: ExplosionParticle(pos, glm::vec2(0, 0), col, 1.2f, ofRandom(16, 32)) {
		float angle = ofRandom(0, TWO_PI);
		float speed = ofRandom(80, 200);
		velocity = glm::vec2(cos(angle), sin(angle)) * speed;
	}

	void update(float dt) override {
		ExplosionParticle::update(dt);
		size += 20 * dt;
	}

	void draw() override {
		ofSetColor(color);
		ofDrawCircle(position, size);
	}
};

class SpiralExplosion : public ExplosionParticle {
	float angle;

public:
	SpiralExplosion(const glm::vec2 & pos, const ofColor & col)
		: ExplosionParticle(pos, glm::vec2(0, 0), col, 1.5f, 8)
		, angle(0) { }

	void update(float dt) override {
		ExplosionParticle::update(dt);
		angle += 10 * dt;
		velocity = glm::vec2(cos(angle), sin(angle)) * 150;
	}

	void draw() override {
		ofSetColor(color);
		ofDrawCircle(position, size);
	}
};

class ofApp : public ofBaseApp {
public:
	void setup();
	void update();
	void draw();
	void mousePressed(int x, int y, int button);
	void keyPressed(int key);

	std::vector<Particle *> particles;
	~ofApp();

private:
	void createRisingParticle();
};

```


#### ofApp.cpp


```

#include "ofApp.h"

void ofApp::setup() {
    ofSetFrameRate(60);
    ofBackground(0);
}

void ofApp::update() {
    float dt = ofGetLastFrameTime();

    for (int i = 0; i < particles.size(); i++) {
        particles[i]->update(dt);
    }

    for (int i = particles.size() - 1; i >= 0; i--) {
        if (particles[i]->shouldExplode()) {
            int explosionType = (int)ofRandom(2);
            int numParticles = (int)ofRandom(20, 30);
            for (int j = 0; j < numParticles; j++) {
                if (explosionType == 0) {
                    particles.push_back(new CircularExplosion(
                        particles[i]->getPosition(), particles[i]->getColor()));
                } else {
                    particles.push_back(new SpiralExplosion(
                        particles[i]->getPosition(), particles[i]->getColor()));
                }
            }
            delete particles[i];
            particles.erase(particles.begin() + i);
        } else if (particles[i]->isDead()) {
            delete particles[i];
            particles.erase(particles.begin() + i);
        }
    }
}

void ofApp::draw() {
    for (int i = 0; i < particles.size(); i++) {
        particles[i]->draw();
    }
}

void ofApp::createRisingParticle() {
    float minX = ofGetWidth() * 0.35f;
    float maxX = ofGetWidth() * 0.65f;
    float spawnX = ofRandom(minX, maxX);
    glm::vec2 pos(spawnX, ofGetHeight());
    glm::vec2 target(ofGetWidth() / 2.0f + ofRandom(-300, 300),
                     ofGetHeight() * 0.10f + ofRandom(-30, 30));
    glm::vec2 direction = glm::normalize(target - pos);
    glm::vec2 vel = direction * ofRandom(250, 350);
    ofColor col;
    col.setHsb(ofRandom(255), 220, 255);
    float lifetime = ofRandom(1.5f, 3.5f);
    particles.push_back(new RisingParticle(pos, vel, col, lifetime));
}

void ofApp::mousePressed(int x, int y, int button) {
    createRisingParticle();
    particles.push_back(new ZigZagParticle(glm::vec2(x, y)));
    particles.push_back(new SpiralParticle(glm::vec2(x, y)));
}

void ofApp::keyPressed(int key) {
    if (key == ' ') {
        for (int i = 0; i < 500; i++) {
            createRisingParticle();
        }
    }
}

ofApp::~ofApp() {
    for (int i = 0; i < particles.size(); i++) {
        delete particles[i];
    }
    particles.clear();
}


```



Agregue 2 tipos de particulas ZigZagParticle y SpiralParticle, ademas añadi la SpiralExplosion.


### Fase 2



#### Evidencia 1 — Herencia en memoria


<img width="1919" height="1016" alt="image" src="https://github.com/user-attachments/assets/488cd303-de54-4aee-be1a-12b268367613" />




##### Elección del punto de inspección: ¿En qué parte del código detuviste la ejecución y por qué elegiste ese punto? La elección del breakpoint es parte del pensamiento crítico y se evalúa.




Elegi el breakpoint en Update donde se recorren las particulas, por que en este momento las particulas fueron creadas y se almacenan en el vector lo que permite ver la memoria y ver como se representa la herencia. 



##### Explicación: ¿Qué variables, valores o estructuras se están observando en la imagen?


Se observa el vector particles, que alamacena punteros de tipo particles pero cada posicion del vector tiene objetos diferentes tipos como RisingParticle o ZigZagParticle que al inspeccionar cada uno se puede ver los atributos como position,velocity o color. 

Ademas de puede ver el puntero _vptr el cual es el que apunta a la tabla de funciones virtuales.



##### Justificación: ¿Cómo demuestra esta captura comprensión del concepto o patrón solicitado?


Por que se ve que un objeto de tipo derivado ZigZagParticle tiene tanto la base particle como sus propios atributos, ademas el uso del puntero _vptr confirma que que el pbjeto tiene informacion sobre su tipo en tiempo de ejecucion y se observa polimorfismo al almacenar diferentes tipos en un solo puntero base, entonces se demuestra que la herencia se elabora un unico bloque en la memoria el cual se hace la clase base y la derivada de la misma 



#### Evidencia 2 — La _vtable de tu nuevo tipo



<img width="1917" height="1018" alt="Captura de pantalla 2026-03-24 201625" src="https://github.com/user-attachments/assets/2848364c-368d-4857-aba5-02580555e7ff" />



##### Elección del punto de inspección: ¿En qué parte del código detuviste la ejecución y por qué elegiste ese punto? La elección del breakpoint es parte del pensamiento crítico y se evalúa.

Aqui puse el breakpoint en el update despues de crear particulas de tipo CircularExplosion al vector particles, y este es el punto en donde el punto ya fue ingresado a la memoria lo qeu se puede ver su estructura interna y la _vtable, y tambien se crearon otro tipo de particulas lo que permite comparar entre varias implementaciones de _vtable



##### Explicación: ¿Qué variables, valores o estructuras se están observando en la imagen?


Se observan el vector particle que tiene varios tipos como ZigZagParticle o CircularExplosion al expandir los objetos se ve su puntero _vtable y se observa dentro del puntero se ve los metodos vurtuales como update, shouldExplode,getPositional y getColor



##### Justificación: ¿Cómo demuestra esta captura comprensión del concepto o patrón solicitado?


Se demuestra el polimorfismo ,ediamte el _vtable aunque todos estan dentro de particles cada tipo de puntero mantiene su propia _vtable, y la diferencia entre cada una tenga su propio comportamiento y sean distintos 



#### Evidencia 3 — Polimorfismo en tiempo de ejecución



<img width="1919" height="1018" alt="image" src="https://github.com/user-attachments/assets/18adf0d0-a92a-4a89-960d-40782328b0f8" />



##### Elección del punto de inspección: ¿En qué parte del código detuviste la ejecución y por qué elegiste ese punto? La elección del breakpoint es parte del pensamiento crítico y se evalúa.



La coloque aqui dado que permite verificar si el programa ejecuta la implementacion especifica de la clase derivada en lugar de la clase base lo que se evidencia el despacho dinamico 



##### Explicación: ¿Qué variables, valores o estructuras se están observando en la imagen?



Esta dentro del meodo update en CircularExplosion y en las variables locales como position, velocity, age, lifetime y size con valores actualizados 



##### Justificación: ¿Cómo demuestra esta captura comprensión del concepto o patrón solicitado?



Se demuestra el polimorfismo por que el metodo update se llama mediante el puntero base particle pero se ejecuta la implementacion especifica de la clase derivada que es CircularExplosion, lo que confirma el uso correcto del dspacho dinamico y las funciones virtuales.



#### Evidencia 4 - Encapsulamiento en el contexto de herencia




<img width="1918" height="1012" alt="Captura de pantalla 2026-03-25 002240" src="https://github.com/user-attachments/assets/5063f338-2f42-4691-8149-3dd8dfa2079e" />


##### Elección del punto de inspección: ¿En qué parte del código detuviste la ejecución y por qué elegiste ese punto? La elección del breakpoint es parte del pensamiento crítico y se evalúa.



Lo coloque en update(float dt) de la clase SpiralExplosion por que se puede ver como la subclase entra a los atributos heradados de la clase base que es ExplosionParticle.



##### Explicación: ¿Qué variables, valores o estructuras se están observando en la imagen?



Se puede ver el objeto actual que es SpiralExplosion y eb el depurador se ven los atributos herados desde ExplosionParticle como position, velocity, color, lifetime y size



##### Justificación: ¿Cómo demuestra esta captura comprensión del concepto o patrón solicitado?



Por que los atributos definidos como protected en la clase base que es visble y puede entrar desde la subclase, mientras que los private no pueden acceder directamente, y se ve en el depurador donde solo se ven lso atributos permitidos



#### Evidencia 5 — Ciclo de vida completo de tu partícula


<img width="1919" height="1017" alt="image" src="https://github.com/user-attachments/assets/329379da-19a2-49f0-a183-44368eb74372" />



##### Elección del punto de inspección: ¿En qué parte del código detuviste la ejecución y por qué elegiste ese punto? La elección del breakpoint es parte del pensamiento crítico y se evalúa.


Se pone el vector a la hora de creacion de la particula y se agrega al vector particles y se puede ver como entra al sistema 


##### Explicación: ¿Qué variables, valores o estructuras se están observando en la imagen?



Se ve el vector particles y el objeto spiralExplosion recien creado y se ven sus atributos como position,velocity,color




##### Justificación: ¿Cómo demuestra esta captura comprensión del concepto o patrón solicitado?


Se evidencia el ciclo de vida del objeto viendo como se instancia dinamicamente y se almacena en el vector




<img width="1916" height="1016" alt="Captura de pantalla 2026-03-26 155111" src="https://github.com/user-attachments/assets/4528d73a-9d5e-4e54-9974-e160d85aab43" />




##### Elección del punto de inspección: ¿En qué parte del código detuviste la ejecución y por qué elegiste ese punto? La elección del breakpoint es parte del pensamiento crítico y se evalúa.



Lo puse ne update dentro del ciclo que recorre particles ya que se puede ver las particulas mientras estan activas 



##### Explicación: ¿Qué variables, valores o estructuras se están observando en la imagen?


Se observa el vector particles con objetos como RisinParticle y ZigzagParticle y se pueden ver sus atributos position,velocity,age  y lifetime y el _vptr



##### Justificación: ¿Cómo demuestra esta captura comprensión del concepto o patrón solicitado?



Se demuestra la etapa donde la particula esta activa y se encuentra en ejecucion y se evidencia como cambia en cada actualizacion.



<img width="1919" height="1011" alt="image" src="https://github.com/user-attachments/assets/2f34de96-113c-4d04-a07f-ec4b0c82f414" />



##### Elección del punto de inspección: ¿En qué parte del código detuviste la ejecución y por qué elegiste ese punto? La elección del breakpoint es parte del pensamiento crítico y se evalúa.


Lo puse en delete particles por que se puede ver como la particula es eliminada 



##### Explicación: ¿Qué variables, valores o estructuras se están observando en la imagen?


Se ve el vector particles con muchos objetos antes de ser eliminado y se ve que se va a ser borrado con delete lo que implica la liberacion de memoria 




##### Justificación: ¿Cómo demuestra esta captura comprensión del concepto o patrón solicitado?


Aqui se demuestra el proceso final de vida del objeto y se ve como el objeto va a ser borrado completando su ciclo de vida 



#### Evidencia 6 — Sin fugas de memoria




<img width="1919" height="1020" alt="Captura de pantalla 2026-03-25 004644" src="https://github.com/user-attachments/assets/1da195f0-2938-4ef9-90a8-25dd4781496a" />




<img width="1108" height="602" alt="image" src="https://github.com/user-attachments/assets/4d26bab7-bad3-430a-bfce-9cdb999df2ba" />



##### Elección del punto de inspección: ¿En qué parte del código detuviste la ejecución y por qué elegiste ese punto? La elección del breakpoint es parte del pensamiento crítico y se evalúa.



Lo coloque en el delete  debido a que es el punto donde se libera la memoria del objeto y tambien en la otra captura en erase para que se verifique que el puntero si es eliminado 



##### Explicación: ¿Qué variables, valores o estructuras se están observando en la imagen?


En la primera captura se observa a particles apunta RisingParticle con una direccion valida lo que indica que aun existe


En la segunda despues de que se ejecute delete y erase se ve a particles disminuye el size osea que se elimino el objeto



##### Justificación: ¿Cómo demuestra esta captura comprensión del concepto o patrón solicitado?



Aqui se demuestra que no hay fugas de memoria por que se libera el objeto mediante delete y se elimina con erase por lo que el tiempo de vida del objeto se libera sin dejar residuos 




#### Evidencia 7 — Prueba de condición límite



<img width="1918" height="1016" alt="image" src="https://github.com/user-attachments/assets/c6bb1096-4bb0-4bc3-9408-f2e59757f0d0" />




##### Elección del punto de inspección: ¿En qué parte del código detuviste la ejecución y por qué elegiste ese punto? La elección del breakpoint es parte del pensamiento crítico y se evalúa.



Se selecciono un escenario donde se generan 500 particulas simultaneamente cuando se presiona espacio donde se presenta un limite donde el sistema maneja una gran cantidad de objetos en la memoria simultaneamente



##### Explicación: ¿Qué variables, valores o estructuras se están observando en la imagen?



Se observa el vector particles que tiene el size=500 y cada uno de estos elementos es un tipo particle en este caso RisingParticle y se puede ver que cada objeto tiene sus atributos como position,velocity, color o lifetime 



##### Justificación: ¿Cómo demuestra esta captura comprensión del concepto o patrón solicitado?


Se puede verificar que el sistema puede manejar gran cantidad de objetos sin fallar, ademas particles crece sin ningun problema manteniedno que cada objeto tenga sus propios atributos 


## Bitácora de reflexión



### Jerarquía de clases del caso de estudio con tus extensiones: muestra el diagrama de clases completo, incluyendo los dos nuevos tipos de partícula que agregaste. Indica qué hereda de qué y qué métodos virtuales hay en cada nivel.



<img width="1253" height="469" alt="image" src="https://github.com/user-attachments/assets/c769889c-0804-4139-869b-96eb0a2a9e3a" />





### Objeto en memoria con herencia y vtable: dibuja cómo se ve en memoria un objeto de uno de tus nuevos tipos. Muestra los campos de cada clase de la jerarquía (base y derivada) y el puntero _vtable apuntando a la tabla de funciones. Anota qué dirección apunta a qué función.



<img width="1134" height="711" alt="Captura de pantalla 2026-03-27 124015" src="https://github.com/user-attachments/assets/3b24ac85-9944-45a7-bf4d-62e24c2f1831" />




### Mecanismo de despacho dinámico (polimorfismo en runtime): dibuja el mecanismo completo: desde la llamada polimórfica (p. ej., particles[i]->update(dt)) hasta la ejecución de la función correcta, pasando por el puntero _vtable y la tabla de funciones. Muestra por qué se ejecuta una versión diferente del método dependiendo del tipo real del objeto.




<img width="877" height="535" alt="image" src="https://github.com/user-attachments/assets/a7ba38f9-d16f-49c9-a418-8901d634b67e" />
