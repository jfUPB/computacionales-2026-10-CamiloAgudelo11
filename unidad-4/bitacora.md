# Unidad 4

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 


### Codigo completo 



#pragma once
#include "ofMain.h"

// Nodo de la cola
struct Node {
    float x, y;
    float radius;
    ofColor color;
    float opacity;
    Node* next;

    Node(float _x, float _y, float _radius, ofColor _color, float _opacity)
        : x(_x), y(_y), radius(_radius), color(_color), opacity(_opacity), next(nullptr) {
    }
};

// Implementación manual de una cola (FIFO)
class BrushQueue {
public:
    Node* front;
    Node* rear;
    int size;
    int maxSize;

    BrushQueue(int _maxSize);
    ~BrushQueue();

    void enqueue(float x, float y, float radius, ofColor color, float opacity);
    void dequeue();
    void clear();
    bool isEmpty();
};

// Constructor
BrushQueue::BrushQueue(int _maxSize) : front(nullptr), rear(nullptr), size(0), maxSize(_maxSize) {}

// Destructor
BrushQueue::~BrushQueue() {
    clear();
}

// enqueue
void BrushQueue::enqueue(float x, float y, float radius, ofColor color, float opacity) {

    Node* newNode = new Node(x, y, radius, color, opacity);

    if (front == nullptr) {
        front = rear = newNode;
    }
    else {
        rear->next = newNode;
        rear = newNode;
    }

    size++;

    if (size > maxSize) {
        dequeue();
    }
}

// dequeue
void BrushQueue::dequeue() {

    if (front == nullptr) return;

    Node* temp = front;
    front = front->next;

    delete temp;

    size--;

    if (front == nullptr) {
        rear = nullptr;
    }
}

// clear
void BrushQueue::clear() {

    while (front != nullptr) {
        dequeue();
    }
}

// isEmpty
bool BrushQueue::isEmpty() {

    return front == nullptr;
}


class ofApp : public ofBaseApp {
public:
    BrushQueue strokes;
    float backgroundHue = 0;

    ofApp() : strokes(50) {}

    void setup();
    void update();
    void draw();
    void keyPressed(int key);
};



ofApp.cpp


#include "ofApp.h"

//--------------------------------------------------------------
void ofApp::setup() {
    ofBackground(0);
}

//--------------------------------------------------------------
void ofApp::update() {

    backgroundHue += 0.2;
    if (backgroundHue > 255) backgroundHue = 0;

    if (ofGetMousePressed()) {

        float x = ofGetMouseX();
        float y = ofGetMouseY();

        float radius = ofRandom(5, 20);

        ofColor color;
        color.setHsb(ofRandom(255), 200, 255);

        float opacity = 200;

        strokes.enqueue(x, y, radius, color, opacity);
    }
}

//--------------------------------------------------------------
void ofApp::draw() {

    ofColor color1, color2;
    color1.setHsb(backgroundHue, 150, 240);
    color2.setHsb(fmod(backgroundHue + 128, 255), 150, 240);
    ofBackgroundGradient(color1, color2, OF_GRADIENT_LINEAR);

    Node* current = strokes.front;

    while (current != nullptr) {

        ofSetColor(current->color, current->opacity);

        ofDrawCircle(current->x, current->y, current->radius);

        current = current->next;
    }
}

//--------------------------------------------------------------
void ofApp::keyPressed(int key) {

    if (key == 'c') {
        strokes.clear();
    }

    if (key == 'a') {

        if (strokes.maxSize == 50)
            strokes.maxSize = 100;
        else
            strokes.maxSize = 50;
    }

    else if (key == 's') {
        ofSaveFrame();
    }
}

## Bitácora de reflexión
