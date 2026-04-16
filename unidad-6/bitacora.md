# Unidad 6



## Bitácora de aplicación 



### Fase 1


#### ofApp.cpp



```
#include "ofApp.h"
#include <algorithm>

void Subject::addObserver(Observer * observer) {
	if (!observer) return;
	if (std::find(observers.begin(), observers.end(), observer) == observers.end()) {
		observers.push_back(observer);
	}
}

void Subject::removeObserver(Observer * observer) {
	if (!observer) return;
	observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end());
}

void Subject::notify(const std::string & event) {
	for (Observer * observer : observers) {
		observer->onNotify(event);
	}
}

Particle::Particle()
	: state(nullptr) {
	position = ofVec2f(ofRandomWidth(), ofRandomHeight());
	velocity = ofVec2f(ofRandom(-0.5f, 0.5f), ofRandom(-0.5f, 0.5f));
	size = ofRandom(2.0f, 5.0f);
	color = ofColor(255);

	state = new NormalState();
	state->onEnter(this);
}

Particle::~Particle() {
	if (state) {
		state->onExit(this);
		delete state;
		state = nullptr;
	}
}

void Particle::setState(State * newState) {
	if (state) {
		state->onExit(this);
		delete state;
	}
	state = newState;
	if (state) {
		state->onEnter(this);
	}
}

void Particle::update() {
	if (state) {
		state->update(this);
	}
	keepInsideWindow();
}

void Particle::draw() {
	ofPushStyle();
	ofSetColor(color);
	ofDrawCircle(position, size);
	ofPopStyle();
}

void Particle::onNotify(const std::string & event) {
	if (event == "attract") {
		setState(new AttractState());
	} else if (event == "repel") {
		setState(new RepelState());
	} else if (event == "stop") {
		setState(new StopState());
	} else if (event == "normal") {
		setState(new NormalState());
	}
	
	else if (event == "chaos") {
		setState(new ChaosState());
	}
}

void Particle::keepInsideWindow() {
	const float W = static_cast<float>(ofGetWidth());
	const float H = static_cast<float>(ofGetHeight());

	if (position.x < 0.0f) {
		position.x = 0.0f;
		velocity.x *= -1.0f;
	} else if (position.x > W) {
		position.x = W;
		velocity.x *= -1.0f;
	}
	if (position.y < 0.0f) {
		position.y = 0.0f;
		velocity.y *= -1.0f;
	} else if (position.y > H) {
		position.y = H;
		velocity.y *= -1.0f;
	}
}

void NormalState::onEnter(Particle * particle) {
	particle->velocity.set(ofRandom(-0.5f, 0.5f), ofRandom(-0.5f, 0.5f));
}

void NormalState::update(Particle * particle) {
	particle->position += particle->velocity;
}

static void steer(Particle * particle, const ofVec2f & toward, float accel, float vmax, float posScale) {
	ofVec2f dir = toward - particle->position;
	float len = dir.length();
	if (len > 1e-6f) {
		dir /= len;
		particle->velocity += dir * accel;
	}
	particle->velocity.limit(vmax);
	particle->position += particle->velocity * posScale;
}

void AttractState::update(Particle * particle) {
	ofVec2f mouse(ofGetMouseX(), ofGetMouseY());
	steer(particle, mouse, 0.05f, 3.0f, 0.2f);
}

void RepelState::update(Particle * particle) {
	ofVec2f mouse(ofGetMouseX(), ofGetMouseY());
	ofVec2f away = particle->position - mouse;
	float len = away.length();
	if (len > 1e-6f) {
		away /= len;
		particle->velocity += away * 0.05f;
	}
	particle->velocity.limit(3.0f);
	particle->position += particle->velocity * 0.2f;
}

void StopState::update(Particle * particle) {
	particle->velocity *= 0.80f;
	if (particle->velocity.lengthSquared() < 1e-4f) {
		particle->velocity.set(0.0f, 0.0f);
	}
	particle->position += particle->velocity;
}


void ChaosState::onEnter(Particle * particle) {
	particle->color = ofColor(255, 255, 0);
}

void ChaosState::update(Particle * particle) {
	particle->velocity += ofVec2f(ofRandom(-1, 1), ofRandom(-1, 1));
	particle->velocity.limit(4.0f);
	particle->position += particle->velocity;
}

Particle * ParticleFactory::createParticle(const std::string & type) {
	Particle * particle = new Particle();
	if (type == "star") {
		particle->size = ofRandom(2.0f, 4.0f);
		particle->color = ofColor(255, 0, 0);
	} else if (type == "shooting_star") {
		particle->size = ofRandom(3.0f, 6.0f);
		particle->color = ofColor(0, 255, 0);
		particle->velocity *= 3.0f;
	} else if (type == "planet") {
		particle->size = ofRandom(5.0f, 8.0f);
		particle->color = ofColor(0, 0, 255);
	}
	
	else if (type == "comet") {
		particle->size = ofRandom(4.0f, 7.0f);
		particle->color = ofColor(255, 255, 0);
		particle->velocity *= 2.5f;
	}
	return particle;
}

ofApp::~ofApp() {
	for (Particle * p : particles) {
		removeObserver(p);
		delete p;
	}
	particles.clear();
}

void ofApp::setup() {
	ofBackground(0);
	particles.reserve(100 + 5 + 10 + 5);

	for (int i = 0; i < 100; ++i) {
		Particle * p = ParticleFactory::createParticle("star");
		particles.push_back(p);
		addObserver(p);
	}
	for (int i = 0; i < 5; ++i) {
		Particle * p = ParticleFactory::createParticle("shooting_star");
		particles.push_back(p);
		addObserver(p);
	}
	for (int i = 0; i < 10; ++i) {
		Particle * p = ParticleFactory::createParticle("planet");
		particles.push_back(p);
		addObserver(p);
	}

	
	for (int i = 0; i < 5; ++i) {
		Particle * p = ParticleFactory::createParticle("comet");
		particles.push_back(p);
		addObserver(p);
	}
}

void ofApp::update() {
	for (Particle * p : particles) {
		p->update();
	}
}

void ofApp::draw() {
	for (Particle * p : particles) {
		p->draw();
	}
}

void ofApp::keyPressed(int key) {
	switch (key) {
	case 's':
		notify("stop");
		break;
	case 'a':
		notify("attract");
		break;
	case 'r':
		notify("repel");
		break;
	case 'n':
		notify("normal");
		break;
	case 'c':
		notify("chaos");
		break; 
	default:
		break;
	}
}

```


#### ofApp.h



```
#pragma once

#include "ofMain.h"
#include <string>
#include <vector>

class Observer {
public:
	virtual ~Observer() = default;
	virtual void onNotify(const std::string & event) = 0;
};

class Subject {
public:
	void addObserver(Observer * observer);
	void removeObserver(Observer * observer);

protected:
	void notify(const std::string & event);

private:
	std::vector<Observer *> observers;
};

class Particle;

class State {
public:
	virtual ~State() = default;
	virtual void update(Particle * particle) = 0;
	virtual void onEnter(Particle * particle) { }
	virtual void onExit(Particle * particle) { }
};

class Particle : public Observer {
public:
	Particle();
	~Particle() override;

	Particle(const Particle &) = delete;
	Particle & operator=(const Particle &) = delete;

	void update();
	void draw();
	void onNotify(const std::string & event) override;

	void setState(State * newState);

	ofVec2f position;
	ofVec2f velocity;
	float size;
	ofColor color;

private:
	void keepInsideWindow();
	State * state;
};

class NormalState : public State {
public:
	void update(Particle * particle) override;
	void onEnter(Particle * particle) override;
};

class AttractState : public State {
public:
	void update(Particle * particle) override;
};

class RepelState : public State {
public:
	void update(Particle * particle) override;
};

class StopState : public State {
public:
	void update(Particle * particle) override;
};


class ChaosState : public State {
public:
	void update(Particle * particle) override;
	void onEnter(Particle * particle) override;
};

class ParticleFactory {
public:
	static Particle * createParticle(const std::string & type);
};

class ofApp : public ofBaseApp, public Subject {
public:
	~ofApp() override;
	void setup() override;
	void update() override;
	void draw() override;
	void keyPressed(int key) override;

private:
	std::vector<Particle *> particles;
};

```


Se agregó un nuevo tipo de partícula llamado "comet" en la ParticleFactory, con mayor velocidad y color diferenciado. Además, se implementó un nuevo estado llamado ChaosState, que modifica el comportamiento de las partículas con movimiento irregular. 



### Fase 2 





#### Evidencia 1 — Tu nueva partícula en la Factory

Demuestra con el depurador que tu nuevo tipo de partícula es creado correctamente por la ParticleFactory. Muestra qué rama del condicional se ejecuta y qué valores tiene el objeto Particle* recién creado (tamaño, color, velocidad).


<img width="1917" height="1015" alt="image" src="https://github.com/user-attachments/assets/11a1e80b-6dd1-4c6a-b12c-d287df39a226" />



##### Eleccion



Lo coloque en el return particle por que se puede ver el estado final del objeto particula antes de que Factory lo entregue al sistema principal lo que garantiza que comet se haya aplicado en la memoria 



##### Explicacion 


SE observa que esta detenido el flujo de ejecucion en el punto de retorno, se observa que el tipo es comet y tambien su direccion de memoria y que posee sus atributos especificos como su tamaño y su color amarillo demostrando que se ejecuta la personalizacion  



##### Justificacion


Se demuestra la implementacion de Factory y como la clase ParticleFactory encapsula la logica de creacion permitiendo que se solicite comet sin conocer sus detalles tecnicos  



#### Evidencia 2 — Tu nuevo estado en la _vtable

Demuestra que tu nuevo estado tiene su propia entrada en la _vtable. Captura la _vtable de una partícula en tu nuevo estado y compárala con la de una partícula en NormalState. ¿Qué entradas cambian? ¿Por qué?




<img width="1919" height="1016" alt="image" src="https://github.com/user-attachments/assets/340bb43b-21c1-47f8-b7e7-d83a161c1c27" />





##### Eleccion


Se coloco aqui el particle ya que es el punto de despacho dinamico donde el programa mira la _vtable para saber que funcion ejecutar 




##### Explicacion 



Se ve el puntero state que esta apuntando a un objeto tipo NormalState y se puede ver su tabla de funciones virtuales con su direccion de memoria especifica 



##### Justificacion


Esto sirve para demostrar que cada objeto tiene su propia tabla en este caso el de NormalState




<img width="1919" height="1003" alt="image" src="https://github.com/user-attachments/assets/225e6998-db8d-4e0a-8d2c-2f70e29ffe8d" />



##### Eleccion



Puse el breakpoint cuando se crea la nueva particula de mi estado y aqui es donde se crea la _vfptr y nos permite verla 



##### Explicacion 



Se ve que ahora state esta el objeto ChaosState y su _vfptr cambio de direccion y se pueden ver las entradas de la tabla apunta ahora a este objeto 



##### Justificacion



Con este cambio se puede ver el polimorfismo de la _vtable y demuestra que state se ejecuta dependiendo de la particula en tiempo de ejecucion 



#### Evidencia 3 — La cadena Observer → State completa

Demuestra la cadena completa cuando se activa tu nuevo estado: desde keyPressed → notify → onNotify → setState. Muestra con el depurador que el nuevo evento llega a onNotify y que el puntero state cambia a tu nuevo tipo de estado.




<img width="1919" height="1011" alt="Captura de pantalla 2026-04-13 162412" src="https://github.com/user-attachments/assets/647a40a9-9a84-4e5c-aedf-8f75feba34a6" />



##### Eleccion



Puse el breakpoint aqui ya que es el punto del destino final de la reaccion de la particula y ver que funciones se invocaron para llegar ahi, pero viendo sus variables locales



##### Explicacion 



Se observa como el newState llega a la funcion setState y en el depurador se observa el objeto ChaosState ademas de ver su _vfptr y se puede ver que las funciones de actualizacion y entrada si apuntan a los de Chaos, demostrando que Observer hace su trabajo notificando llegando a onNotify 



##### Justificacion



Se demuestra como se materializa el final de la cadena de eventos conectandose con el patron State y confirma que onNotify fue exitoso ya que instancia un nuevo objeto en este caso ChaosState y pasarlo por referencia a la particula, lo que lo deja preparado para cambiar el comportamiento en el siguiente ciclo de dibujo  



<img width="1916" height="1016" alt="image" src="https://github.com/user-attachments/assets/b112b322-1bf8-4790-8df2-7256b888f5e6" />




##### Eleccion


Aqui se mantiene el mismo breakpoint y por la misma razon pero viendo las pilas de llamadas 



##### Explicacion 



Se demuestra como se materializa el final de la cadena de eventos conectandose con el patron State y confirma que onNotify fue exitoso ya que instancia un nuevo objeto en este caso ChaosState y pasarlo por referencia a la particula, lo que lo deja preparado para cambiar el comportamiento en el siguiente ciclo de dibujo  





##### Justificacion



Se demuestra la cadena de comunicacion del patrn observer y es la misma particula la que escucha el mensaje de onNotify y reacciona haciendo el cambio  



#### Evidencia 4 — Decisión de diseño justificada

Elige UNA decisión de diseño que hayas tomado al extender el código (por ejemplo: por qué tu nuevo estado hereda directamente de State y no de un estado existente; o por qué configuraste la partícula de cierta manera en la factory). Demuestra con el depurador la consecuencia concreta en memoria o comportamiento de esa decisión, y justifica por qué fue la mejor opción.



<img width="1917" height="1019" alt="image" src="https://github.com/user-attachments/assets/8c95e80f-a7c9-4437-bad1-e594c48ef668" />




##### Eleccion


Se coloco ahi el breakpoint debido a que representa a un tipo de aduana que tiene el sistema y aqui es donde Factory recibe una cadena de texto y decide con polimorfismo como debe nacer ese nuevo objeto antes de llegar al sistema principal 



##### Explicacion 



Se detuvo en el tipo comet donde en el panel de variables se confirma que en type es igual a la nueva extension del sistema, ademas se observa el nuevo objeto particle ya se instancio en la memoria y esta a punto de recibir sus atributos que lo diferencia de la star o planet 




##### Justificacion



Esta decision de diseño de usar Factory es mejor que configurar cada particula directamente en el setup, ya que es mas facil por ejemplo cambiar algun color solo modifico esa linea de Factory y no cada lugar donde se crea esta particula permitiendo que el codigo sea mas sencillo




## Bitácora de reflexión



Usando excalidraw, construye los siguientes gráficos:

### Los tres patrones colaborando: dibuja el sistema completo con tus extensiones. Muestra Observer, Factory y State como capas de responsabilidad separadas y las flechas de comunicación entre ellas. ¿Qué responsabilidad tiene cada patrón? ¿Dónde comienza y termina cada uno?


<img width="929" height="880" alt="image" src="https://github.com/user-attachments/assets/57bf83b2-9376-42b1-8189-9204d491b3ac" />



### State y la _vtable — conexión con la unidad anterior: en la unidad 5 estudiaste cómo la _vtable implementa el polimorfismo. Ahora conecta eso con el patrón State. Cuando una partícula llama a setState, ¿Cambia su _vtable? ¿Por qué sí o por qué no? Dibuja la respuesta.


<img width="1415" height="442" alt="image" src="https://github.com/user-attachments/assets/32be7936-e4f0-44b6-8e57-07d2f92c5baa" />



### ¿Por qué estos patrones?: para cada uno de los tres patrones, completa la frase:

 “Usé el patrón [X] porque sin él tendría que [problema concreto] y eso causaría [consecuencia de diseño].”



Use el patron Observer por que sin el tendria que recorrer manualmente todas las particulas y cambiarlas de estado directamente y eso causaria alto acoplamiento 



Usé el patrón State porque sin él tendría que usar múltiples condicionales dentro de Particle para definir su comportamiento, y eso causaría código difícil de mantener y extender.



Usé el patrón Factory porque sin él tendría que crear manualmente cada tipo de partícula en múltiples partes del código, y eso causaría duplicación y falta de control.



Reflexiona también sobre estas preguntas (no hay respuestas incorrectas):



### ¿Qué decisión de diseño de esta unidad cambiarías ahora que la entiendes mejor?


Se podrian usar smart points para evitar errores




### ¿En qué otro contexto, un juego, una app, un sistema que conozcas, usarías alguno de estos tres patrones?



Se podria usar en videojuegos de como los enemigos reaccionan a un evento del jugador 



### ¿Qué pregunta de la sustentación te costó más responder, y por qué crees que fue así?


