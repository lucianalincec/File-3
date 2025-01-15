let orbs = [];
let cosmicDust = [];
let cosmicDustCount = 4000; // Número de partículas cósmicas
let orbCount = 20; // Número de círculos distribuidos por el canvas
let walls = [];
let particle;

function setup() {
  createCanvas(1000, 1000);
  noCursor();

  // Crear círculos distribuidos aleatoriamente
  for (let i = 0; i < orbCount; i++) {
    let x = random(width);
    let y = random(height);
    let r = random(50, 130); // Radio aleatorio entre 50 y 130
    let speed = random(0.005, 0.02); // Velocidad de movimiento
    let angle = random(TWO_PI); // Ángulo inicial aleatorio
    orbs.push({ x, y, r, speed, angle });
  }

  // Crear partículas de polvo cósmico
  for (let i = 0; i < cosmicDustCount; i++) {
    cosmicDust.push(createVector(random(width), random(height)));
  }

  particle = new Particle();
}

function draw() {
  background(0);

  // Dibujar partículas de polvo cósmico
  for (let dust of cosmicDust) {
    fill(200, 200, 255, 50);
    noStroke();
    ellipse(dust.x, dust.y, 1);
  }

  walls = []; // Limpiar paredes antes de actualizar

  // Actualizar posiciones de los círculos y crear sus límites
  for (let orb of orbs) {
    orb.angle += orb.speed; // Actualizar ángulo
    let x = orb.x + 30 * cos(orb.angle); // Pequeño movimiento orbital
    let y = orb.y + 30 * sin(orb.angle);

    // Crear límites del círculo
    let numPoints = 100; // Puntos para aproximar el círculo
    let angleStep = TWO_PI / numPoints;
    for (let i = 0; i < numPoints; i++) {
      let x1 = x + orb.r * cos(i * angleStep);
      let y1 = y + orb.r * sin(i * angleStep);
      let x2 = x + orb.r * cos((i + 1) * angleStep);
      let y2 = y + orb.r * sin((i + 1) * angleStep);
      walls.push(new Boundary(x1, y1, x2, y2));
    }

    // Dibujar el círculo
    noFill();
    stroke(100, 100, 255, 150);
    ellipse(x, y, orb.r * 2);
  }

  for (let wall of walls) {
    wall.show();
  }

  // Actualizar y mostrar la partícula exploradora
  particle.update(mouseX, mouseY);
  particle.show();
  particle.explore(walls);
}

class Boundary {
  constructor(x1, y1, x2, y2) {
    this.a = createVector(x1, y1);
    this.b = createVector(x2, y2);
  }

  show() {
    stroke(100, 100, 255, 100);
    line(this.a.x, this.a.y, this.b.x, this.b.y);
  }
}

class Particle {
  constructor() {
    this.pos = createVector(width / 2, height / 2);
  }

  update(x, y) {
    this.pos.x = x || this.pos.x;
    this.pos.y = y || this.pos.y;
  }

  explore(walls) {
    for (let wall of walls) {
      let intersect = this.checkCollision(wall);
      if (intersect) {
        stroke(255, 150, 50);
        line(this.pos.x, this.pos.y, intersect.x, intersect.y);
      }
    }
  }

  checkCollision(wall) {
    const x1 = wall.a.x;
    const y1 = wall.a.y;
    const x2 = wall.b.x;
    const y2 = wall.b.y;

    const x3 = this.pos.x;
    const y3 = this.pos.y;
    const x4 = x3 + random(-2, 2);
    const y4 = y3 + random(-2, 2);

    const den = (x1 - x2) * (y3 - y4) - (y1 - y2) * (x3 - x4);
    if (den == 0) {
      return;
    }

    const t = ((x1 - x3) * (y3 - y4) - (y1 - y3) * (x3 - x4)) / den;
    const u = -((x1 - x2) * (y1 - y3) - (y1 - y2)) / den;
    if (t > 0 && t < 1 && u > 0) {
      const pt = createVector();
      pt.x = x1 + t * (x2 - x1);
      pt.y = y1 + t * (y2 - y1);
      return pt;
    }
  }

  show() {
    fill(255, 200, 100);
    noStroke();
    ellipse(this.pos.x, this.pos.y, 5);
  }
}

