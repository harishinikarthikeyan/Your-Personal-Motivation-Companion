export class Simulation {
    constructor(width, height) {
        this.width = width;
        this.height = height;
        this.particles = [];

        // Define adjustable parameters
        this.params = {
            count: { value: 100, min: 0, max: 1000, step: 10, label: 'Entity Count' },
            speed: { value: 2.5, min: 0.1, max: 10, step: 0.1, label: 'Speed Factor' },
            connectionRadius: { value: 100, min: 0, max: 300, step: 10, label: 'Link Radius' },
            repulsion: { value: 0.5, min: 0, max: 2, step: 0.1, label: 'Repulsion' }
        };

        this.reset();
    }

    resize(w, h) {
        this.width = w;
        this.height = h;
    }

    updateParam(key, value) {
        if (this.params[key]) {
            this.params[key].value = parseFloat(value);
            if (key === 'count') {
                this.adjustCount();
            }
        }
    }

    reset() {
        this.particles = [];
        this.adjustCount();
    }

    adjustCount() {
        const target = this.params.count.value;
        const current = this.particles.length;

        if (current < target) {
            for (let i = 0; i < target - current; i++) {
                this.particles.push(this.createParticle());
            }
        } else if (current > target) {
            this.particles.splice(target);
        }
    }

    createParticle() {
        return {
            x: Math.random() * this.width,
            y: Math.random() * this.height,
            vx: (Math.random() - 0.5) * 2,
            vy: (Math.random() - 0.5) * 2,
            color: `hsl(${Math.random() * 60 + 200}, 70%, 60%)`, // Blue-Purple range
            size: Math.random() * 3 + 2
        };
    }

    update(dt) {
        const speed = this.params.speed.value;

        for (let p of this.particles) {

            // Mouse Repulsion (if we had mouse input, skipping for simplicity but preparing logic)
            // ...

            // Simple movement
            p.x += p.vx * speed;
            p.y += p.vy * speed;

            // Bounce off walls
            if (p.x < 0 || p.x > this.width) {
                p.vx *= -1;
                p.x = Math.max(0, Math.min(this.width, p.x));
            }
            if (p.y < 0 || p.y > this.height) {
                p.vy *= -1;
                p.y = Math.max(0, Math.min(this.height, p.y));
            }
        }
    }

    draw(ctx) {
        ctx.clearRect(0, 0, this.width, this.height);

        const r = this.params.connectionRadius.value;
        const rSq = r * r;

        // Draw connections (optimized: only close ones)
        ctx.strokeStyle = 'rgba(56, 189, 248, 0.15)';
        ctx.lineWidth = 1;
        ctx.beginPath();

        // Simple O(N^2) for demo purposes (fine for < 500 particles)
        // For production, use a quadtree or spatial hash
        if (r > 0) {
            for (let i = 0; i < this.particles.length; i++) {
                const p1 = this.particles[i];
                for (let j = i + 1; j < this.particles.length; j++) {
                    const p2 = this.particles[j];
                    const dx = p1.x - p2.x;
                    const dy = p1.y - p2.y;
                    if (dx * dx + dy * dy < rSq) {
                        ctx.moveTo(p1.x, p1.y);
                        ctx.lineTo(p2.x, p2.y);
                    }
                }
            }
        }
        ctx.stroke();

        // Draw particles
        for (let p of this.particles) {
            ctx.fillStyle = p.color;
            ctx.beginPath();
            ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
            ctx.fill();
        }
    }

    getCount() {
        return this.particles.length;
    }
}
