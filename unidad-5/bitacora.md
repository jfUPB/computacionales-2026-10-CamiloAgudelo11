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


### Fase 2



#### Evidencia 1 — Herencia en memoria


<img width="1919" height="1016" alt="image" src="https://github.com/user-attachments/assets/488cd303-de54-4aee-be1a-12b268367613" />


#### Evidencia 2 — La _vtable de tu nuevo tipo



<img width="1917" height="1018" alt="Captura de pantalla 2026-03-24 201625" src="https://github.com/user-attachments/assets/2848364c-368d-4857-aba5-02580555e7ff" />


#### Evidencia 3 — Polimorfismo en tiempo de ejecución


<img width="1919" height="1018" alt="image" src="https://github.com/user-attachments/assets/18adf0d0-a92a-4a89-960d-40782328b0f8" />



#### Evidencia 4 - Encapsulamiento en el contexto de herencia




<img width="1918" height="1012" alt="Captura de pantalla 2026-03-25 002240" src="https://github.com/user-attachments/assets/5063f338-2f42-4691-8149-3dd8dfa2079e" />



#### Evidencia 5 — Ciclo de vida completo de tu partícula


<img width="1919" height="1017" alt="image" src="https://github.com/user-attachments/assets/329379da-19a2-49f0-a183-44368eb74372" />


#### Evidencia 6 — Sin fugas de memoria




<img width="1919" height="1020" alt="Captura de pantalla 2026-03-25 004644" src="https://github.com/user-attachments/assets/1da195f0-2938-4ef9-90a8-25dd4781496a" />




<img width="1919" height="1015" alt="Captura de pantalla 2026-03-25 004947" src="https://github.com/user-attachments/assets/4f8ac17c-3d46-4333-a5db-a168689e2dd4" />




#### Evidencia 7 — Prueba de condición límite



<img width="1918" height="1016" alt="image" src="https://github.com/user-attachments/assets/c6bb1096-4bb0-4bc3-9408-f2e59757f0d0" />


## Bitácora de reflexión
