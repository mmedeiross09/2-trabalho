const printableChars = Array.from({ length: 95 }, (_, index) => String.fromCharCode(index + 32));

const textInput = document.getElementById('textInput');
const keyInput = document.getElementById('keyInput');
const resultOutput = document.getElementById('resultOutput');
const encryptBtn = document.getElementById('encryptBtn');
const decryptBtn = document.getElementById('decryptBtn');
const demoBtn = document.getElementById('demoBtn');
const traceList = document.getElementById('traceList');

function normalizeKey() {
  const value = Number.parseInt(keyInput.value, 10);
  if (!Number.isFinite(value) || value < 1) {
    keyInput.value = '1';
    return 1;
  }

  const normalized = value % printableChars.length;
  keyInput.value = String(normalized === 0 ? printableChars.length : normalized);
  return normalized === 0 ? printableChars.length : normalized;
}

function getCharIndex(char) {
  return printableChars.indexOf(char);
}

function formatTraceStep(step) {
  return `${step.original} → índice ${step.index} + chave ${step.key} = ${step.newIndex} → ${step.transformed}`;
}

function transformText(text, key, mode) {
  let transformed = '';
  const trace = [];

  for (const char of text) {
    const currentIndex = getCharIndex(char);

    if (currentIndex === -1) {
      transformed += char;
      trace.push({
        original: char,
        index: 'não está na tabela',
        key,
        newIndex: 'mantido',
        transformed: char,
      });
      continue;
    }

    const effectiveShift = mode === 'encrypt' ? key : -key;
    const newIndex = (currentIndex + effectiveShift + printableChars.length) % printableChars.length;
    const transformedChar = printableChars[newIndex];

    transformed += transformedChar;
    trace.push({
      original: char,
      index: currentIndex,
      key,
      newIndex,
      transformed: transformedChar,
    });
  }

  return { transformed, trace };
}

function updateTraceList(trace) {
  if (!trace || trace.length === 0) {
    traceList.innerHTML = '<li>Nenhuma etapa para mostrar.</li>';
    return;
  }

  traceList.innerHTML = trace
    .slice(0, 12)
    .map((step) => `<li>${formatTraceStep(step)}</li>`)
    .join('');
}

function handleEncrypt() {
  const value = textInput.value;
  const key = normalizeKey();
  const { transformed, trace } = transformText(value, key, 'encrypt');

  resultOutput.textContent = transformed;
  updateTraceList(trace);
}

function handleDecrypt() {
  const value = textInput.value;
  const key = normalizeKey();
  const { transformed, trace } = transformText(value, key, 'decrypt');

  resultOutput.textContent = transformed;
  updateTraceList(trace);
}

function loadDemo() {
  const demoText = 'Senha@123';
  const demoKey = 7;

  textInput.value = demoText;
  keyInput.value = String(demoKey);

  const { transformed, trace } = transformText(demoText, demoKey, 'encrypt');
  resultOutput.textContent = transformed;
  updateTraceList(trace);
}

encryptBtn.addEventListener('click', handleEncrypt);
decryptBtn.addEventListener('click', handleDecrypt);
demoBtn.addEventListener('click', loadDemo);

loadDemo();
