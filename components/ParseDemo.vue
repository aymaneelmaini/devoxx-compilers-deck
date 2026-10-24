<script setup lang="ts">
import { computed, ref } from 'vue'

type Token = { type: 'num' | 'op' | 'lparen' | 'rparen'; text: string }
type Expr =
  | { kind: 'num'; value: number }
  | { kind: 'bin'; op: string; left: Expr; right: Expr }
  | { kind: 'group'; expr: Expr }

const source = ref('1 + 2 * 3')

function tokenize(src: string): Token[] {
  const tokens: Token[] = []
  let i = 0
  while (i < src.length) {
    const c = src[i]
    if (/\s/.test(c)) { i++; continue }
    if (/\d/.test(c)) {
      let j = i
      while (j < src.length && /\d/.test(src[j])) j++
      tokens.push({ type: 'num', text: src.slice(i, j) })
      i = j
      continue
    }
    if ('+-*/'.includes(c)) { tokens.push({ type: 'op', text: c }); i++; continue }
    if (c === '(') { tokens.push({ type: 'lparen', text: c }); i++; continue }
    if (c === ')') { tokens.push({ type: 'rparen', text: c }); i++; continue }
    throw new Error(`Unexpected character '${c}' at position ${i}`)
  }
  return tokens
}

function parse(tokens: Token[]): Expr {
  let pos = 0
  const peek = () => tokens[pos]
  const next = () => tokens[pos++]
  const peekOp = (...ops: string[]) => peek()?.type === 'op' && ops.includes(peek().text)

  // lowest precedence: + and -
  function term(): Expr {
    let expr = factor()
    while (peekOp('+', '-')) {
      const op = next().text
      expr = { kind: 'bin', op, left: expr, right: factor() }
    }
    return expr
  }
  // higher precedence: * and /
  function factor(): Expr {
    let expr = primary()
    while (peekOp('*', '/')) {
      const op = next().text
      expr = { kind: 'bin', op, left: expr, right: primary() }
    }
    return expr
  }
  function primary(): Expr {
    const t = next()
    if (!t) throw new Error('Unexpected end of input')
    if (t.type === 'num') return { kind: 'num', value: Number(t.text) }
    if (t.type === 'lparen') {
      const expr = term()
      if (next()?.type !== 'rparen') throw new Error("Expected ')'")
      return { kind: 'group', expr }
    }
    throw new Error(`Unexpected token '${t.text}'`)
  }

  const tree = term()
  if (pos < tokens.length) throw new Error(`Unexpected token '${peek().text}'`)
  return tree
}

function evaluate(e: Expr): number {
  if (e.kind === 'num') return e.value
  if (e.kind === 'group') return evaluate(e.expr)
  const l = evaluate(e.left)
  const r = evaluate(e.right)
  if (e.op === '+') return l + r
  if (e.op === '-') return l - r
  if (e.op === '*') return l * r
  return l / r
}

function drawTree(e: Expr, prefix = '', isLast = true, isRoot = true): string[] {
  const label = e.kind === 'num' ? String(e.value) : e.kind === 'group' ? '( )' : e.op
  const children: Expr[] = e.kind === 'num' ? [] : e.kind === 'group' ? [e.expr] : [e.left, e.right]
  const lines = [isRoot ? label : prefix + (isLast ? '└─ ' : '├─ ') + label]
  const childPrefix = isRoot ? '' : prefix + (isLast ? '   ' : '│  ')
  children.forEach((c, i) => lines.push(...drawTree(c, childPrefix, i === children.length - 1, false)))
  return lines
}

function codeRepr(e: Expr, indent = 0): string {
  const pad = '  '.repeat(indent)
  if (e.kind === 'num') return String(e.value)
  if (e.kind === 'group') return `(${codeRepr(e.expr, indent)})`
  return `${codeRepr(e.left, indent)} ${e.op} ${codeRepr(e.right, indent)}`
}

const output = computed(() => {
  try {
    const tokens = tokenize(source.value)
    const ast = parse(tokens)
    return {
      tokens,
      code: codeRepr(ast),
      tree: drawTree(ast).join('\n'),
      value: String(evaluate(ast)),
      error: '',
    }
  } catch (err) {
    return { tokens: [] as Token[], code: '', tree: '', value: '', error: (err as Error).message }
  }
})
</script>

<template>
  <div class="w-full max-w-3xl">
    <div class="text-sm opacity-60 font-mono mb-2">Expression</div>
    <input
      v-model="source"
      spellcheck="false"
      aria-label="Expression"
      class="w-full bg-transparent border-b-2 border-current outline-none font-mono text-3xl py-2"
    />
    <div v-if="!output.error" class="mt-6 text-3xl font-700">
      = {{ output.value }}
    </div>
    <div v-else class="mt-6 text-xl opacity-80 text-red-500">
      {{ output.error }}
    </div>
    <div class="mt-8">
      <div class="text-sm opacity-60 font-mono mb-2">Code Structure</div>
      <pre class="text-base leading-tight font-mono bg-opacity-10 bg-gray-500 p-3 rounded mb-4">{{ output.code }}</pre>
      <div class="text-sm opacity-60 font-mono mb-2">Tree Visualization</div>
      <pre class="text-lg leading-tight font-mono border-l-2 border-opacity-30 border-current pl-4">{{ output.tree }}</pre>
    </div>
    <div class="mt-6 text-xs opacity-50 font-mono">
      tokens: {{ output.tokens.map((t) => t.text).join('  ') || '—' }}
    </div>
  </div>
</template>
