<script lang="ts">
  import { fly } from "svelte/transition";

  const DIMENSIONE_MASSIMA = 20 * 1024 * 1024; // 20 MB

  let file = $state<File | null>(null);
  let errore = $state<string | null>(null);
  let trascinando = $state(false);

  let eImmagine = $derived(file?.type.startsWith("image/") ?? false);
  let tipoLeggibile = $derived(eImmagine ? "Immagine" : "PDF");

  // Crea un indirizzo temporaneo per mostrare l'anteprima della foto
  let anteprima = $derived(file && eImmagine ? URL.createObjectURL(file) : null);

  // Quando l'anteprima cambia o il componente sparisce, libera la memoria
  $effect(() => {
    const url = anteprima;
    return () => {
      if (url) URL.revokeObjectURL(url);
    };
  });

  function formattaDimensione(byte: number): string {
    const numero = new Intl.NumberFormat("it-IT", { maximumFractionDigits: 1 });
    if (byte < 1024) return `${byte} B`;
    if (byte < 1024 * 1024) return `${numero.format(byte / 1024)} KB`;
    return `${numero.format(byte / (1024 * 1024))} MB`;
  }

  function controllaFile(scelto: File) {
    const ePdf = scelto.type === "application/pdf";
    const eFoto = scelto.type.startsWith("image/");

    if (!ePdf && !eFoto) {
      file = null;
      errore = "Formato non supportato: carica un PDF o un'immagine.";
      return;
    }

    if (scelto.size > DIMENSIONE_MASSIMA) {
      file = null;
      errore = `Il file è troppo grande (${formattaDimensione(scelto.size)}). Il massimo è ${formattaDimensione(DIMENSIONE_MASSIMA)}.`;
      return;
    }

    file = scelto;
    errore = null;
  }

  function gestisciFile(e: Event & { currentTarget: HTMLInputElement }) {
    const scelto = e.currentTarget.files?.[0];
    if (!scelto) return;
    controllaFile(scelto);
    e.currentTarget.value = "";
  }

  function gestisciDragOver(e: DragEvent) {
    e.preventDefault();
    trascinando = true;
  }

  function gestisciDragLeave() {
    trascinando = false;
  }

  function gestisciDrop(e: DragEvent) {
    e.preventDefault();
    trascinando = false;
    const scelto = e.dataTransfer?.files?.[0];
    if (!scelto) return;
    controllaFile(scelto);
  }

  function rimuoviFile() {
    file = null;
    errore = null;
  }
</script>

<div class="caricatore">
  <label
    class="area"
    class:trascinando
    ondragover={gestisciDragOver}
    ondragleave={gestisciDragLeave}
    ondrop={gestisciDrop}
  >
    <input
      class="nascosto"
      type="file"
      accept=".pdf,image/*"
      onchange={gestisciFile}
    />

    <span class="bolla" aria-hidden="true">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <path d="M12 16V4" />
        <path d="M7 9l5-5 5 5" />
        <path d="M4 16v3a1 1 0 0 0 1 1h14a1 1 0 0 0 1-1v-3" />
      </svg>
    </span>

    <span class="titolo">
      {trascinando ? "Rilascia il file qui" : "Trascina qui il tuo referto"}
    </span>
    <span class="oppure">oppure</span>
    <span class="pulsante">Scegli un file</span>
    <span class="suggerimento">PDF o foto del foglio · massimo 20 MB</span>
  </label>

  {#if errore}
    <div class="errore" role="alert" transition:fly={{ y: -6, duration: 200 }}>
      <svg aria-hidden="true" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round">
        <circle cx="12" cy="12" r="9" />
        <path d="M12 8v5" />
        <path d="M12 16h.01" />
      </svg>
      <span>{errore}</span>
    </div>
  {/if}

  {#if file}
    <div class="scheda" transition:fly={{ y: 8, duration: 220 }}>
      {#if anteprima}
        <img class="miniatura" src={anteprima} alt="Anteprima del referto" />
      {:else}
        <span class="icona-file" aria-hidden="true">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M14 3H7a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V8z" />
            <path d="M14 3v5h5" />
            <path d="M9 13h6" />
            <path d="M9 17h6" />
          </svg>
        </span>
      {/if}

      <div class="info">
        <span class="nome">{file.name}</span>
        <span class="dettagli">{tipoLeggibile} · {formattaDimensione(file.size)}</span>
      </div>

      <button class="rimuovi" type="button" onclick={rimuoviFile} aria-label="Rimuovi il file">
        <svg aria-hidden="true" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round">
          <path d="M6 6l12 12" />
          <path d="M18 6L6 18" />
        </svg>
      </button>
    </div>
  {/if}
</div>

<style>
  /* Tutti i colori in un unico posto */
  .caricatore {
    --accento: #0d9488;
    --accento-forte: #0f766e;
    --accento-tenue: #f0fdfa;
    --testo: #1f2937;
    --testo-tenue: #6b7280;
    --bordo: #d1d5db;
    --sfondo: #ffffff;
    --sfondo-area: #f9fafb;
    --errore: #b91c1c;
    --errore-tenue: #fef2f2;

    max-width: 560px;
    margin: 0 auto;
    color: var(--testo);
  }

  @media (prefers-color-scheme: dark) {
    .caricatore {
      --accento: #2dd4bf;
      --accento-forte: #5eead4;
      --accento-tenue: #0f2f2c;
      --testo: #f3f4f6;
      --testo-tenue: #9ca3af;
      --bordo: #374151;
      --sfondo: #1f2937;
      --sfondo-area: #111827;
      --errore: #fca5a5;
      --errore-tenue: #3b1212;
    }
  }

  /* Area di caricamento */
  .area {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.4rem;
    padding: 2.5rem 1.5rem;
    border: 2px dashed var(--bordo);
    border-radius: 16px;
    background: var(--sfondo-area);
    text-align: center;
    cursor: pointer;
    transition: border-color 0.2s, background-color 0.2s;
  }

  .area:hover,
  .area.trascinando {
    border-color: var(--accento);
    background: var(--accento-tenue);
  }

  .area.trascinando {
    border-style: solid;
  }

  .area:focus-within {
    outline: 3px solid var(--accento);
    outline-offset: 3px;
  }

  .area > span {
    pointer-events: none;
  }

  .bolla {
    display: grid;
    place-items: center;
    width: 56px;
    height: 56px;
    margin-bottom: 0.5rem;
    border-radius: 50%;
    background: var(--accento-tenue);
    color: var(--accento);
    transition: transform 0.2s;
  }

  .bolla svg {
    width: 26px;
    height: 26px;
  }

  .area.trascinando .bolla {
    transform: scale(1.15) translateY(-4px);
  }

  .titolo {
    font-size: 1.1rem;
    font-weight: 600;
  }

  .oppure {
    font-size: 0.85rem;
    color: var(--testo-tenue);
  }

  .pulsante {
    margin-top: 0.25rem;
    padding: 0.55rem 1.2rem;
    border-radius: 999px;
    background: var(--accento);
    color: var(--sfondo);
    font-weight: 500;
    transition: background-color 0.2s;
  }

  .area:hover .pulsante {
    background: var(--accento-forte);
  }

  .suggerimento {
    margin-top: 0.75rem;
    font-size: 0.85rem;
    color: var(--testo-tenue);
  }

  .nascosto {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip: rect(0 0 0 0);
    white-space: nowrap;
  }

  /* Messaggio di errore */
  .errore {
    display: flex;
    align-items: center;
    gap: 0.6rem;
    margin-top: 1rem;
    padding: 0.75rem 1rem;
    border: 1px solid var(--errore);
    border-radius: 10px;
    background: var(--errore-tenue);
    color: var(--errore);
    font-size: 0.95rem;
  }

  .errore svg {
    flex-shrink: 0;
    width: 20px;
    height: 20px;
  }

  /* Scheda del file caricato */
  .scheda {
    display: flex;
    align-items: center;
    gap: 0.9rem;
    margin-top: 1rem;
    padding: 0.75rem;
    border: 1px solid var(--bordo);
    border-radius: 12px;
    background: var(--sfondo);
  }

  .miniatura,
  .icona-file {
    flex-shrink: 0;
    width: 48px;
    height: 48px;
    border-radius: 8px;
  }

  .miniatura {
    object-fit: cover;
  }

  .icona-file {
    display: grid;
    place-items: center;
    background: var(--accento-tenue);
    color: var(--accento);
  }

  .icona-file svg {
    width: 24px;
    height: 24px;
  }

  .info {
    display: flex;
    flex-direction: column;
    flex: 1;
    min-width: 0;
  }

  .nome {
    font-weight: 500;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .dettagli {
    font-size: 0.85rem;
    color: var(--testo-tenue);
  }

  .rimuovi {
    display: grid;
    place-items: center;
    flex-shrink: 0;
    width: 36px;
    height: 36px;
    border: none;
    border-radius: 50%;
    background: transparent;
    color: var(--testo-tenue);
    cursor: pointer;
    transition: background-color 0.2s, color 0.2s;
  }

  .rimuovi:hover {
    background: var(--errore-tenue);
    color: var(--errore);
  }

  .rimuovi svg {
    width: 18px;
    height: 18px;
  }
</style>