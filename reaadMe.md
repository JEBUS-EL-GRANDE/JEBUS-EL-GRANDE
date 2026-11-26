import { useEffect, useRef, useState, type ChangeEvent } from "react";
import { createFileRoute } from "@tanstack/react-router";
import { TerminalBanner } from "@/components/TerminalBanner";
import {
  DEFAULT_ASCII_ART,
  DEFAULT_BANNER,
  exportBannerPng,
  imageFileToAscii,
  type BannerData,
} from "@/lib/ascii";
import { buildReadmeSnippet } from "@/lib/readme";

const STORAGE_KEY = "termbanner:v3";

export const Route = createFileRoute("/")({
  head: () => ({
    meta: [
      { title: "JEBUS — Banner estilo terminal" },
      {
        name: "description",
        content: "Banner de perfil estilo terminal oscura para Juan Cordero (JEBUS) con editor y descarga en PNG.",
      },
      { property: "og:title", content: "JEBUS — Banner estilo terminal" },
      {
        property: "og:description",
        content: "Banner de perfil estilo terminal oscura con arte ASCII interactivo y exportación en PNG.",
      },
      { property: "og:type", content: "website" },
      { name: "twitter:card", content: "summary_large_image" },
    ],
  }),
  component: Index,
});

function Index() {
  const [banner, setBanner] = useState<BannerData>(DEFAULT_BANNER);
  const [hydrated, setHydrated] = useState(false);
  const [uploadError, setUploadError] = useState<string | null>(null);
  const [copied, setCopied] = useState(false);
  const fileInputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    try {
      const saved = window.localStorage.getItem(STORAGE_KEY);
      if (saved) {
        const parsed = JSON.parse(saved) as Partial<BannerData>;
        setBanner({ ...DEFAULT_BANNER, ...parsed });
      }
    } catch {
      // Ignore malformed localStorage data
    }
    setHydrated(true);
  }, []);

  useEffect(() => {
    if (!hydrated) return;
    try {
      window.localStorage.setItem(STORAGE_KEY, JSON.stringify(banner));
    } catch {
      // Ignore quota errors
    }
  }, [banner, hydrated]);

  const updateField = (field: keyof BannerData, value: string) => {
    setBanner((prev) => ({ ...prev, [field]: value }));
  };

  const copyReadmeSnippet = async () => {
    const snippet = buildReadmeSnippet(banner);
    try {
      await navigator.clipboard.writeText(snippet);
      setCopied(true);
      window.setTimeout(() => setCopied(false), 2200);
    } catch {
      setUploadError("Tu navegador no dejó copiar el código. Cópialo a mano desde el recuadro.");
    }
  };

  const handleImageUpload = async (e: ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;
    if (!file.type.startsWith("image/")) {
      setUploadError("Selecciona un archivo de imagen válido (PNG, JPG o WebP).");
      return;
    }
    setUploadError(null);
    try {
      const ascii = await imageFileToAscii(file);
      updateField("asciiArt", ascii);
    } catch (err) {
      setUploadError(err instanceof Error ? err.message : "No se pudo convertir la imagen.");
    } finally {
      e.target.value = "";
    }
  };

  return (
    <main className="mx-auto flex min-h-screen w-full max-w-5xl flex-col justify-center gap-8 px-4 py-10 sm:px-6">
      <header className="flex flex-wrap items-center justify-between gap-4">
        <div>
          <h1 className="text-xl font-semibold tracking-tight text-foreground sm:text-2xl">
            Banner estilo terminal
          </h1>
          <p className="text-sm text-muted-foreground">
            Edita los campos o sube una foto para el retrato ASCII y descárgalo en PNG (1500×540).
          </p>
        </div>

        <div className="flex flex-wrap items-center gap-2.5">
          <input
            ref={fileInputRef}
            type="file"
            accept="image/*"
            onChange={handleImageUpload}
            className="hidden"
          />
          <button
            type="button"
            onClick={() => fileInputRef.current?.click()}
            className="inline-flex items-center gap-2 rounded-md border border-border bg-secondary px-3.5 py-2 text-sm font-medium text-secondary-foreground transition-colors hover:bg-accent"
          >
            Subir foto ASCII
          </button>
          <button
            type="button"
            onClick={() => exportBannerPng(banner)}
            className="inline-flex items-center gap-2 rounded-md bg-primary px-4 py-2 text-sm font-medium text-primary-foreground transition-opacity hover:opacity-90"
          >
            Descargar PNG
          </button>
          <button
            type="button"
            onClick={copyReadmeSnippet}
            className="inline-flex items-center gap-2 rounded-md border border-border bg-secondary px-3.5 py-2 text-sm font-medium text-secondary-foreground transition-colors hover:bg-accent"
          >
            {copied ? "¡Copiado!" : "Copiar código README"}
          </button>
        </div>
      </header>

      <TerminalBanner data={banner} />

      {uploadError && (
        <p role="alert" className="text-sm text-destructive">
          {uploadError}
        </p>
      )}

      <section aria-label="Personalizar contenido" className="grid grid-cols-1 gap-4 rounded-xl border border-border bg-card p-5 sm:grid-cols-2 lg:grid-cols-3">
        <label className="space-y-1.5 text-xs text-muted-foreground">
          <span>Usuario / ruta</span>
          <input
            className="editor-input"
            value={banner.handle}
            onChange={(e) => updateField("handle", e.target.value)}
            maxLength={32}
          />
        </label>

        <label className="space-y-1.5 text-xs text-muted-foreground">
          <span>Nombre</span>
          <input
            className="editor-input"
            value={banner.name}
            onChange={(e) => updateField("name", e.target.value)}
            maxLength={40}
          />
        </label>

        <label className="space-y-1.5 text-xs text-muted-foreground">
          <span>Ubicación</span>
          <input
            className="editor-input"
            value={banner.location}
            onChange={(e) => updateField("location", e.target.value)}
            maxLength={40}
          />
        </label>

        <label className="space-y-1.5 text-xs text-muted-foreground sm:col-span-2 lg:col-span-3">
          <span>About — línea 1</span>
          <input
            className="editor-input"
            value={banner.aboutLine1}
            onChange={(e) => updateField("aboutLine1", e.target.value)}
            maxLength={90}
          />
        </label>

        <label className="space-y-1.5 text-xs text-muted-foreground sm:col-span-2 lg:col-span-3">
          <span>About — línea 2 (opcional)</span>
          <input
            className="editor-input"
            value={banner.aboutLine2}
            onChange={(e) => updateField("aboutLine2", e.target.value)}
            maxLength={90}
          />
        </label>

        <label className="space-y-1.5 text-xs text-muted-foreground sm:col-span-2 lg:col-span-3">
          <span>Stack</span>
          <input
            className="editor-input"
            value={banner.stack}
            onChange={(e) => updateField("stack", e.target.value)}
            maxLength={110}
          />
        </label>

        <label className="space-y-1.5 text-xs text-muted-foreground sm:col-span-2 lg:col-span-3">
          <span>Elsewhere (enlaces)</span>
          <input
            className="editor-input"
            value={banner.elsewhere}
            onChange={(e) => updateField("elsewhere", e.target.value)}
            maxLength={110}
          />
        </label>

        {banner.asciiArt !== DEFAULT_ASCII_ART && (
          <div className="sm:col-span-2 lg:col-span-3">
            <button
              type="button"
              onClick={() => updateField("asciiArt", DEFAULT_ASCII_ART)}
              className="text-xs text-muted-foreground underline underline-offset-4 hover:text-foreground"
            >
              Restaurar arte ASCII original
            </button>
          </div>
        )}
      </section>

      <section
        aria-label="Código para tu README"
        className="rounded-xl border border-border bg-card p-5"
      >
        <h2 className="text-sm font-semibold text-foreground">Pégalo en tu README.md</h2>
        <p className="mt-1 text-xs text-muted-foreground">
          Sube el PNG descargado a tu repositorio de perfil y pega esta línea tal cual.
        </p>
        <pre className="mt-3 overflow-x-auto rounded-lg border border-border bg-input px-4 py-3 font-mono text-xs break-all text-terminal-ink">
          {buildReadmeSnippet(banner)}
        </pre>
      </section>
    </main>
  );
}
