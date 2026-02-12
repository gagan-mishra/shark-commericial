# Ripple Effect Snippet

## JavaScript (React - `src/App.jsx`)
```jsx
const TARGET_CELL_SIZE = 120
const PASTEL_COLORS = ['#4a2f2f', '#4a3b1d', '#24405c', '#244a36', '#4a2f1a', '#35244a']

const timeoutsRef = useRef([])
const prefersReduced = useRef(false)
const rippleActiveRef = useRef(false)

const [grid, setGrid] = useState({
  rows: 0,
  cols: 0,
  cellWidth: TARGET_CELL_SIZE,
  cellHeight: TARGET_CELL_SIZE,
})
const [flashes, setFlashes] = useState({})
const [transitionMs, setTransitionMs] = useState(150)

const hexToRgba = (hex, alpha) => {
  const normalized = hex.replace('#', '')
  const bigint = parseInt(normalized, 16)
  const r = (bigint >> 16) & 255
  const g = (bigint >> 8) & 255
  const b = bigint & 255
  return `rgba(${r}, ${g}, ${b}, ${alpha})`
}

const pickRandomColor = () => PASTEL_COLORS[Math.floor(Math.random() * PASTEL_COLORS.length)]

const schedule = (fn, delay) => {
  const id = setTimeout(fn, delay)
  timeoutsRef.current.push(id)
}

const clearTimers = () => {
  timeoutsRef.current.forEach(clearTimeout)
  timeoutsRef.current = []
}

const handleClick = (event) => {
  if (event.target.closest?.('.content-block, form, input, textarea, button, select, a, [role="button"], [onclick]')) return
  if (rippleActiveRef.current) return

  const { cols, rows } = grid
  if (!cols || !rows || !gridRef.current) return

  const rect = gridRef.current.getBoundingClientRect()
  const clickX = event.clientX - rect.left
  const clickY = event.clientY - rect.top

  clearTimers()
  setFlashes({})

  const baseColor = pickRandomColor()
  const maxDistance = Math.hypot(cols * TARGET_CELL_SIZE, rows * TARGET_CELL_SIZE)
  const waveSpeed = TARGET_CELL_SIZE / 40
  const bandWidth = TARGET_CELL_SIZE * 2
  const bandDuration = bandWidth / waveSpeed
  const alpha = 0.8
  const totalDuration = maxDistance / waveSpeed + bandDuration + 200
  setTransitionMs(bandDuration / 2)

  rippleActiveRef.current = true
  schedule(() => {
    rippleActiveRef.current = false
  }, totalDuration)

  for (let r = 0; r < rows; r += 1) {
    for (let c = 0; c < cols; c += 1) {
      const centerX = (c + 0.5) * TARGET_CELL_SIZE
      const centerY = (r + 0.5) * TARGET_CELL_SIZE
      const distance = Math.hypot(centerX - clickX, centerY - clickY)
      const color = hexToRgba(baseColor, alpha)
      const startDelay = distance / waveSpeed - bandDuration / 2
      const endDelay = distance / waveSpeed + bandDuration / 2
      const key = `${r}-${c}`

      schedule(() => {
        setFlashes((prev) => ({ ...prev, [key]: color }))
      }, Math.max(0, startDelay))

      schedule(() => {
        setFlashes((prev) => {
          const next = { ...prev }
          delete next[key]
          return next
        })
      }, Math.max(0, endDelay))
    }
  }
}

// Render (inside your grid loop)
// const cellClass = `cell${isFlashing ? ' flash' : ''}`
// <div className={cellClass} style={flashColor ? { '--flash-color': flashColor } : undefined} />
```

## CSS (Grid + Ripple - `src/App.css`)
```css
.grid {
  position: fixed;
  inset: 0;
  display: grid;
  pointer-events: auto;
  z-index: 0;
  gap: 0;
  overflow: hidden;
  background: var(--grid-base);
}

.cell {
  background: var(--grid-cell);
  box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0.08);
  transition-property: background-color, box-shadow, transform;
  transition-timing-function: ease;
  transition-duration: 300ms;
  will-change: background-color;
  backface-visibility: hidden;
  transform: translateZ(0);
  overflow: hidden;
}

.cell.flash {
  background: var(--flash-color, var(--accent));
  box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0);
  animation: rippleScale 600ms ease forwards;
}

@keyframes rippleScale {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.35);
  }
  70% {
    transform: scale(0.9);
  }
  100% {
    transform: scale(1);
  }
}
```
