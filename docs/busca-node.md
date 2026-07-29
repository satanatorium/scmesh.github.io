# Buscar nodeId

Se o seu nó não estiver na lista e você deseja publicar essa informação aqui no site para pesquisa, por favor, clique [aqui](https://forms.gle/r8BgrbnS4D5isq7Z6) e preencha os dados no formulário.  

Consulte os dados publicados de um nó Meshtastic pelo seu `nodeId`. 

<div class="node-lookup" markdown="0">
  <form id="node-lookup-form" class="node-lookup__form">
    <label for="node-id">nodeId</label>
    <div class="node-lookup__controls">
      <input id="node-id" name="nodeId" type="search" placeholder="Ex.: !a1b2c3d4" autocomplete="off" required>
      <button type="submit">Buscar</button>
    </div>
  </form>

  <p id="node-lookup-status" class="node-lookup__status" role="status" aria-live="polite"></p>
  <section id="node-lookup-result" class="node-lookup__result" hidden aria-live="polite"></section>
</div>

<style>
  .node-lookup { max-width: 42rem; margin: 1.5rem 0; }
  .node-lookup__form label { display: block; font-weight: 700; margin-bottom: .45rem; }
  .node-lookup__controls { display: flex; gap: .65rem; }
  .node-lookup__controls input { flex: 1; min-width: 0; }
  .node-lookup__status { min-height: 1.5rem; margin: 1rem 0 .5rem; }
  .node-lookup__result { border-left: .2rem solid var(--md-primary-fg-color); padding: .8rem 1rem; background: var(--md-code-bg-color); }
  .node-lookup__result dl { display: grid; grid-template-columns: minmax(8rem, 35%) 1fr; gap: .45rem .8rem; margin: 0; }
  .node-lookup__result dt { font-weight: 700; }
  .node-lookup__result dd { margin: 0; overflow-wrap: anywhere; }
  @media (max-width: 30rem) { .node-lookup__controls { flex-direction: column; } }
</style>

<script>
  (() => {
    const form = document.getElementById('node-lookup-form');
    const input = document.getElementById('node-id');
    const status = document.getElementById('node-lookup-status');
    const result = document.getElementById('node-lookup-result');
    const formatKey = (key) => key.replace(/([a-z])([A-Z])/g, '$1 $2').replace(/[_-]+/g, ' ').replace(/^./, (character) => character.toUpperCase());
    const displayValue = (value) => value === null || value === undefined ? '—' : typeof value === 'object' ? JSON.stringify(value) : String(value);

    const showNode = (nodes) => {
      result.replaceChildren();
      nodes.forEach((node) => {
        const title = document.createElement('h2');
        title.textContent = node.nodeId || node.id || 'Nó encontrado';
        result.appendChild(title);
        const details = document.createElement('dl');
        Object.entries(node).forEach(([key, value]) => {
          const term = document.createElement('dt');
          term.textContent = formatKey(key);
          const description = document.createElement('dd');
          description.textContent = displayValue(value);
          details.append(term, description);
        });
        result.appendChild(details);
        result.hidden = false;s
      })
    };

    const findNode = (data, nodeId) => {
      if (Array.isArray(data)) return data.filter((node) => String(node?.nodeId ?? node?.id ?? '').trim().includes(nodeId));
      if (data && typeof data === 'object') {
        const key = Object.keys(data).filter((item) => item.trim().toLowerCase().includes(nodeId));
        if (key) return typeof data[key] === 'object' ? { nodeId: key, ...data[key] } : { nodeId: key, value: data[key] };
      }
      return undefined;
    };

    form.addEventListener('submit', async (event) => {
      event.preventDefault();
      const nodeId = input.value.trim().toLowerCase();
      if (!nodeId) return;
      result.hidden = true;
      status.textContent = 'Consultando dados…';
      try {
        const response = await fetch('../data/nodes.json', { cache: 'no-cache' });
        if (!response.ok) throw new Error('data unavailable');
        const node = findNode(await response.json(), nodeId);
        if (!node) { status.textContent = 'Nenhum nó encontrado para "' + input.value.trim() + '".'; return; }
        status.textContent = 'Nó encontrado.';
        showNode(node);
      } catch (error) {
        status.textContent = 'Não foi possível carregar os dados dos nós. Tente novamente mais tarde.';
        console.log(error);
      }
    });
  })();
</script>
