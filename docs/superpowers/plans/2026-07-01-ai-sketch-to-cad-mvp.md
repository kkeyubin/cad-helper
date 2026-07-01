# AI Sketch to CAD MVP Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Web MVP where a user uploads a hand-drawn sketch, confirms the detected CAD scenario, answers targeted clarification questions, previews a vector demo, and downloads an editable DXF.

**Architecture:** Use a Next.js TypeScript app with a small domain core that owns the intermediate CAD model, scenario packs, question generation, validation, preview geometry, and DXF export. Keep AI behind an adapter interface so the MVP can run with a deterministic mock analyzer and then switch to an OpenAI vision analyzer without changing the UI or domain logic.

**Tech Stack:** Node.js via nvm, Next.js App Router, React, TypeScript, Vitest, Testing Library, Playwright, OpenAI Responses API for image understanding, hand-written ASCII DXF exporter for 2D entities.

---

## Scope Check

The product spec describes a broad platform, but the MVP is one working vertical slice:

1. One uploaded image.
2. Scenario classification with a first-class municipal pipeline/survey scene pack.
3. Clarification questions with variable options, custom text answers, numeric inputs, and skip support.
4. A structured intermediate CAD model.
5. A browser preview generated from that model.
6. A DXF download generated from that model.

The MVP keeps multi-industry support as an extension point through scene packs. It does not implement DWG, BIM, STEP, IFC, persistent accounts, multi-user collaboration, or production-grade construction drawing standards.

## External References

- OpenAI Responses API reference: https://platform.openai.com/docs/api-reference/responses/create
- OpenAI Images and Vision guide: https://platform.openai.com/docs/guides/images-vision

Use the documented `input_image` content item with a data URL and `response.output_text` for the OpenAI analyzer. Keep `OPENAI_MODEL` configurable because model names change faster than the product architecture.

## File Structure

Create or modify these files:

- Create: `.nvmrc`  
  Pins the Node major version used by nvm.
- Modify: `.gitignore`  
  Adds local build outputs and environment files.
- Create: `package.json`  
  Defines scripts, dependencies, and engines.
- Create: `tsconfig.json`  
  TypeScript settings for Next.js and tests.
- Create: `vitest.config.ts`  
  Unit test configuration.
- Create: `playwright.config.ts`  
  Browser test configuration.
- Create: `src/test/setup.ts`  
  Test environment setup.
- Create: `tests/smoke.test.ts`  
  Confirms the test runner is wired before domain code exists.
- Create: `src/domain/types.ts`  
  Canonical intermediate CAD model types.
- Create: `src/domain/sampleModel.ts`  
  Deterministic sample project used by tests and mock AI.
- Create: `src/domain/scenarioPacks.ts`  
  Scenario pack registry and municipal scene pack.
- Create: `src/domain/classifyScene.ts`  
  Scene candidate ranking based on extracted text and simple visual hints.
- Create: `src/domain/questions.ts`  
  Clarification question generation and prioritization.
- Create: `src/domain/answers.ts`  
  Applies user answers back into the intermediate model.
- Create: `src/domain/validation.ts`  
  Readiness checks and unresolved issue reporting.
- Create: `src/domain/preview.ts`  
  Converts the intermediate model into SVG-friendly preview primitives.
- Create: `src/domain/dxf.ts`  
  Converts the intermediate model into editable ASCII DXF.
- Create: `src/ai/analyzer.ts`  
  AI provider interface and deterministic mock analyzer.
- Create: `src/ai/openaiAnalyzer.ts`  
  OpenAI vision analyzer implementation.
- Create: `src/server/projects.ts`  
  In-memory project store for the MVP.
- Create: `src/app/api/analyze/route.ts`  
  Upload endpoint that creates the first model and question queue.
- Create: `src/app/api/projects/[id]/scene/route.ts`  
  Scene confirmation endpoint that writes the chosen scene back to the project.
- Create: `src/app/api/projects/[id]/answer/route.ts`  
  Answer endpoint that updates the model and next questions.
- Create: `src/app/api/projects/[id]/export/route.ts`  
  DXF download endpoint.
- Create: `src/app/layout.tsx`  
  App shell metadata and global CSS import.
- Create: `src/app/page.tsx`  
  Main upload, clarification, preview, and export workflow.
- Create: `src/components/FileUpload.tsx`  
  Sketch upload control.
- Create: `src/components/SceneConfirmation.tsx`  
  Scenario selection with custom input.
- Create: `src/components/QuestionPanel.tsx`  
  Variable option, numeric, text, and skip answer UI.
- Create: `src/components/PreviewCanvas.tsx`  
  SVG vector demo renderer.
- Create: `src/components/InspectorPanel.tsx`  
  Object list, unresolved issues, and layer toggles.
- Create: `src/components/ExportPanel.tsx`  
  Final readiness and DXF download actions.
- Create: `src/styles/globals.css`  
  App styling.
- Create: `tests/domain/*.test.ts`  
  Unit tests for the domain core.
- Create: `tests/e2e/mvp.spec.ts`  
  End-to-end workflow test.

## Implementation Rules

- Use `nvm` for Node because the local environment uses nvm-managed `node` and `npm`.
- Do not use the network proxy unless dependency installation or GitHub access is blocked.
- Keep the domain core independent from React and Next.js.
- Keep the AI response as a suggestion; the domain model and validator decide whether the project is ready.
- Generate DXF from `CadProject`, never directly from the uploaded image.
- Commit after each task that leaves tests passing.

## Task 1: Scaffold The Web App

**Files:**
- Create: `.nvmrc`
- Modify: `.gitignore`
- Create: `package.json`
- Create: `tsconfig.json`
- Create: `vitest.config.ts`
- Create: `playwright.config.ts`
- Create: `src/test/setup.ts`
- Create: `tests/smoke.test.ts`
- Create: `src/app/layout.tsx`
- Create: `src/app/page.tsx`
- Create: `src/styles/globals.css`

- [ ] **Step 1: Pin Node for nvm**

Create `.nvmrc`:

```text
22
```

- [ ] **Step 2: Expand ignored local files**

Update `.gitignore` so it contains exactly:

```gitignore
.DS_Store
node_modules
.next
dist
coverage
test-results
playwright-report
.env
.env.local
```

- [ ] **Step 3: Create package manifest**

Create `package.json`:

```json
{
  "name": "cad-helper",
  "private": true,
  "version": "0.1.0",
  "type": "module",
  "engines": {
    "node": ">=22"
  },
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "test": "vitest run",
    "test:watch": "vitest",
    "e2e": "playwright test"
  },
  "dependencies": {
    "@nanoid/non-secure": "^5.1.5",
    "next": "^15.3.0",
    "openai": "^5.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "zod": "^3.25.0"
  },
  "devDependencies": {
    "@playwright/test": "^1.53.0",
    "@testing-library/jest-dom": "^6.6.0",
    "@testing-library/react": "^16.3.0",
    "@types/node": "^22.15.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "@vitejs/plugin-react": "^4.5.0",
    "jsdom": "^26.1.0",
    "typescript": "^5.8.0",
    "vitest": "^3.2.0"
  }
}
```

- [ ] **Step 4: Install dependencies**

Run:

```bash
nvm use
npm install
```

Expected: `npm install` exits with code 0 and creates `package-lock.json`.

- [ ] **Step 5: Add TypeScript config**

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["dom", "dom.iterable", "es2022"],
    "allowJs": false,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "src/**/*.ts", "src/**/*.tsx", "tests/**/*.ts", "tests/**/*.tsx"],
  "exclude": ["node_modules"]
}
```

- [ ] **Step 6: Add test configs**

Create `vitest.config.ts`:

```ts
import react from "@vitejs/plugin-react";
import { defineConfig } from "vitest/config";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: ["./src/test/setup.ts"],
    include: ["tests/**/*.test.ts", "tests/**/*.test.tsx"]
  },
  resolve: {
    alias: {
      "@": new URL("./src", import.meta.url).pathname
    }
  }
});
```

Create `playwright.config.ts`:

```ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests/e2e",
  webServer: {
    command: "npm run dev",
    url: "http://127.0.0.1:3000",
    reuseExistingServer: !process.env.CI
  },
  use: {
    baseURL: "http://127.0.0.1:3000",
    trace: "on-first-retry"
  },
  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] }
    }
  ]
});
```

Create `src/test/setup.ts`:

```ts
import "@testing-library/jest-dom/vitest";
```

Create `tests/smoke.test.ts`:

```ts
import { describe, expect, it } from "vitest";

describe("test runner", () => {
  it("runs Vitest in the CAD helper workspace", () => {
    expect("cad-helper").toContain("cad");
  });
});
```

- [ ] **Step 7: Add minimal app shell**

Create `src/app/layout.tsx`:

```tsx
import type { Metadata } from "next";
import "@/styles/globals.css";

export const metadata: Metadata = {
  title: "CAD Helper",
  description: "AI-assisted sketch to CAD confirmation workflow"
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="zh-CN">
      <body>{children}</body>
    </html>
  );
}
```

Create `src/app/page.tsx`:

```tsx
export default function HomePage() {
  return (
    <main className="app-shell">
      <section className="workspace">
        <h1>CAD Helper</h1>
        <p>上传手绘草图，确认场景，回答澄清问题，预览并导出 DXF。</p>
      </section>
    </main>
  );
}
```

Create `src/styles/globals.css`:

```css
:root {
  color-scheme: light;
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  background: #f5f7fa;
  color: #17202a;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

button,
input,
select,
textarea {
  font: inherit;
}

.app-shell {
  min-height: 100vh;
  padding: 24px;
}

.workspace {
  max-width: 1280px;
  margin: 0 auto;
}
```

- [ ] **Step 8: Run scaffold checks**

Run:

```bash
npm run test
npm run build
```

Expected: smoke test passes with code 0; build completes with code 0.

- [ ] **Step 9: Commit scaffold**

```bash
git add .nvmrc .gitignore package.json package-lock.json tsconfig.json vitest.config.ts playwright.config.ts src/test/setup.ts tests/smoke.test.ts src/app/layout.tsx src/app/page.tsx src/styles/globals.css
git commit -m "chore: scaffold CAD helper web app"
```

## Task 2: Define The Intermediate CAD Model

**Files:**
- Create: `src/domain/types.ts`
- Create: `src/domain/sampleModel.ts`
- Test: `tests/domain/types.test.ts`

- [ ] **Step 1: Write failing model test**

Create `tests/domain/types.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { sampleMunicipalProject } from "@/domain/sampleModel";

describe("CadProject model", () => {
  it("stores scenario, entities, constraints, questions, answers, and versions", () => {
    expect(sampleMunicipalProject.project.scene).toBe("municipal_pipeline_survey");
    expect(sampleMunicipalProject.entities.map((entity) => entity.type)).toContain("building");
    expect(sampleMunicipalProject.constraints[0].type).toBe("distance");
    expect(sampleMunicipalProject.questions).toEqual([]);
    expect(sampleMunicipalProject.answers).toEqual([]);
    expect(sampleMunicipalProject.versions[0].reason).toBe("initial mock extraction");
  });
});
```

- [ ] **Step 2: Verify the test fails**

Run:

```bash
npm run test -- tests/domain/types.test.ts
```

Expected: FAIL because `@/domain/sampleModel` does not exist.

- [ ] **Step 3: Add canonical domain types**

Create `src/domain/types.ts`:

```ts
export type ScenarioId =
  | "municipal_pipeline_survey"
  | "building_floor_plan"
  | "cabinet_furniture"
  | "mechanical_part"
  | "electrical_wiring"
  | "custom";

export type ProjectStatus = "uploaded" | "scene_confirming" | "clarifying" | "preview_ready" | "export_ready";
export type CadUnit = "m" | "cm" | "mm";
export type ConfirmationState = "confirmed" | "needs_user_confirmation" | "skipped";

export type EntityType =
  | "baseline"
  | "road"
  | "pipeline"
  | "waterway"
  | "boundary"
  | "building"
  | "node"
  | "text"
  | "dimension"
  | "unknown";

export type Geometry =
  | { kind: "point"; x: number; y: number }
  | { kind: "line"; points: Array<{ x: number; y: number }> }
  | { kind: "rectangle"; x: number; y: number; width: number; height: number }
  | { kind: "text"; x: number; y: number; value: string };

export interface CadEntity {
  id: string;
  type: EntityType;
  label: string;
  geometry: Geometry;
  layer: string;
  confidence: number;
  confirmation: ConfirmationState;
  sourceText?: string;
}

export type ConstraintType = "distance" | "unit" | "parallel" | "perpendicular" | "touching" | "unknown";

export interface CadConstraint {
  id: string;
  type: ConstraintType;
  value?: number;
  unit?: CadUnit;
  appliesTo: string[];
  confidence: number;
  confirmation: ConfirmationState;
}

export type QuestionKind = "single_choice" | "number" | "text" | "geometry_relation" | "scene_choice";

export interface QuestionOption {
  id: string;
  label: string;
  value: string;
}

export interface ClarificationQuestion {
  id: string;
  kind: QuestionKind;
  priority: number;
  prompt: string;
  targetIds: string[];
  options: QuestionOption[];
  allowCustomAnswer: boolean;
  allowSkip: boolean;
  unit?: CadUnit;
}

export interface ClarificationAnswer {
  id: string;
  questionId: string;
  value: string;
  customValue?: string;
  numericValue?: number;
  unit?: CadUnit;
  skipped: boolean;
  createdAt: string;
}

export interface ProjectVersion {
  id: string;
  reason: string;
  createdAt: string;
}

export interface SceneCandidate {
  id: ScenarioId;
  label: string;
  confidence: number;
  reason: string;
}

export interface CadProject {
  project: {
    id: string;
    scene: ScenarioId;
    unit: CadUnit;
    status: ProjectStatus;
    sourceImageName?: string;
    sceneCandidates: SceneCandidate[];
  };
  entities: CadEntity[];
  constraints: CadConstraint[];
  questions: ClarificationQuestion[];
  answers: ClarificationAnswer[];
  versions: ProjectVersion[];
}
```

- [ ] **Step 4: Add sample model**

Create `src/domain/sampleModel.ts`:

```ts
import type { CadProject } from "./types";

export const sampleMunicipalProject: CadProject = {
  project: {
    id: "project_sample_municipal",
    scene: "municipal_pipeline_survey",
    unit: "m",
    status: "clarifying",
    sourceImageName: "sample-field-sketch.jpg",
    sceneCandidates: [
      {
        id: "municipal_pipeline_survey",
        label: "市政/道路/管线/外业测绘草图",
        confidence: 0.86,
        reason: "图中包含沿线距离、建筑名、水沟和检查井类标注。"
      }
    ]
  },
  entities: [
    {
      id: "entity_baseline_001",
      type: "baseline",
      label: "沿线基准线",
      geometry: {
        kind: "line",
        points: [
          { x: 40, y: 210 },
          { x: 760, y: 210 }
        ]
      },
      layer: "baseline",
      confidence: 0.82,
      confirmation: "needs_user_confirmation",
      sourceText: "连续距离标注所在的主线"
    },
    {
      id: "entity_building_001",
      type: "building",
      label: "邮政大厦",
      geometry: {
        kind: "rectangle",
        x: 690,
        y: 120,
        width: 120,
        height: 52
      },
      layer: "building",
      confidence: 0.72,
      confirmation: "needs_user_confirmation",
      sourceText: "邮政大厦 50米 17米"
    },
    {
      id: "entity_waterway_001",
      type: "waterway",
      label: "水沟",
      geometry: {
        kind: "line",
        points: [
          { x: 60, y: 310 },
          { x: 250, y: 320 },
          { x: 480, y: 318 },
          { x: 740, y: 300 }
        ]
      },
      layer: "water",
      confidence: 0.66,
      confirmation: "needs_user_confirmation",
      sourceText: "水沟/沟渠波浪线"
    }
  ],
  constraints: [
    {
      id: "constraint_distance_001",
      type: "distance",
      value: 50,
      unit: "m",
      appliesTo: ["entity_building_001"],
      confidence: 0.7,
      confirmation: "needs_user_confirmation"
    }
  ],
  questions: [],
  answers: [],
  versions: [
    {
      id: "version_001",
      reason: "initial mock extraction",
      createdAt: "2026-07-01T00:00:00.000Z"
    }
  ]
};
```

- [ ] **Step 5: Run and pass model test**

Run:

```bash
npm run test -- tests/domain/types.test.ts
```

Expected: PASS.

- [ ] **Step 6: Commit model**

```bash
git add src/domain/types.ts src/domain/sampleModel.ts tests/domain/types.test.ts
git commit -m "feat: add intermediate CAD model"
```

## Task 3: Add Scenario Packs And Scene Classification

**Files:**
- Create: `src/domain/scenarioPacks.ts`
- Create: `src/domain/classifyScene.ts`
- Test: `tests/domain/scenarioPacks.test.ts`
- Test: `tests/domain/classifyScene.test.ts`

- [ ] **Step 1: Write failing scenario pack test**

Create `tests/domain/scenarioPacks.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { getScenarioPack, listScenarioPacks } from "@/domain/scenarioPacks";

describe("scenario packs", () => {
  it("includes the municipal pipeline survey scene pack", () => {
    const pack = getScenarioPack("municipal_pipeline_survey");

    expect(pack.label).toBe("市政/道路/管线/外业测绘草图");
    expect(pack.entityTypes).toContain("building");
    expect(pack.defaultLayers).toContain("待确认");
    expect(listScenarioPacks().map((candidate) => candidate.id)).toContain("custom");
  });
});
```

- [ ] **Step 2: Write failing scene classifier test**

Create `tests/domain/classifyScene.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { classifySceneFromText } from "@/domain/classifyScene";

describe("classifySceneFromText", () => {
  it("ranks municipal survey first when sketch text contains wells, buildings, and meter distances", () => {
    const candidates = classifySceneFromText("邮政大厦 雨水检查井 50米 17米 水沟 124米");

    expect(candidates[0].id).toBe("municipal_pipeline_survey");
    expect(candidates[0].confidence).toBeGreaterThan(0.75);
    expect(candidates.at(-1)?.id).toBe("custom");
  });
});
```

- [ ] **Step 3: Verify tests fail**

Run:

```bash
npm run test -- tests/domain/scenarioPacks.test.ts tests/domain/classifyScene.test.ts
```

Expected: FAIL because the scenario modules do not exist.

- [ ] **Step 4: Implement scenario packs**

Create `src/domain/scenarioPacks.ts`:

```ts
import type { EntityType, ScenarioId } from "./types";

export interface ScenarioPack {
  id: ScenarioId;
  label: string;
  description: string;
  entityTypes: EntityType[];
  defaultUnits: string[];
  defaultLayers: string[];
  keywords: string[];
}

const packs: ScenarioPack[] = [
  {
    id: "municipal_pipeline_survey",
    label: "市政/道路/管线/外业测绘草图",
    description: "外业手绘的道路、管线、水沟、检查井、建筑和沿线距离示意图。",
    entityTypes: ["baseline", "road", "pipeline", "waterway", "boundary", "building", "node", "text", "dimension", "unknown"],
    defaultUnits: ["m", "cm", "mm"],
    defaultLayers: ["基准线", "道路", "管线", "水系", "建筑", "节点", "尺寸", "文字", "待确认"],
    keywords: ["米", "井", "水沟", "雨水", "污水", "道路", "管线", "大厦", "民房", "检查井"]
  },
  {
    id: "building_floor_plan",
    label: "建筑/户型/装修平面图",
    description: "室内墙体、门窗、房间、梁柱、面积和层高草图。",
    entityTypes: ["boundary", "building", "text", "dimension", "unknown"],
    defaultUnits: ["mm", "cm", "m"],
    defaultLayers: ["墙体", "门窗", "房间", "尺寸", "文字", "待确认"],
    keywords: ["卧室", "客厅", "厨房", "卫生间", "门", "窗", "墙"]
  },
  {
    id: "cabinet_furniture",
    label: "柜体/家具定制图",
    description: "板件、孔位、封边、五金、材料和厚度草图。",
    entityTypes: ["boundary", "dimension", "node", "text", "unknown"],
    defaultUnits: ["mm", "cm"],
    defaultLayers: ["板件", "孔位", "五金", "尺寸", "文字", "待确认"],
    keywords: ["柜", "板", "孔", "封边", "铰链", "拉手", "厚"]
  },
  {
    id: "mechanical_part",
    label: "机械零件草图",
    description: "孔径、倒角、公差、中心线、视图和材料草图。",
    entityTypes: ["boundary", "dimension", "node", "text", "unknown"],
    defaultUnits: ["mm"],
    defaultLayers: ["轮廓", "孔", "中心线", "尺寸", "文字", "待确认"],
    keywords: ["孔径", "倒角", "公差", "中心线", "R", "Φ", "材料"]
  },
  {
    id: "electrical_wiring",
    label: "电气/弱电布线图",
    description: "开关、插座、回路、桥架、线缆和配电箱草图。",
    entityTypes: ["pipeline", "node", "text", "dimension", "unknown"],
    defaultUnits: ["m", "mm"],
    defaultLayers: ["回路", "桥架", "线缆", "设备", "尺寸", "文字", "待确认"],
    keywords: ["开关", "插座", "回路", "桥架", "线缆", "配电箱"]
  },
  {
    id: "custom",
    label: "其他，请输入",
    description: "用户输入的自定义场景。",
    entityTypes: ["unknown", "text", "dimension"],
    defaultUnits: ["m", "cm", "mm"],
    defaultLayers: ["默认", "文字", "尺寸", "待确认"],
    keywords: []
  }
];

export function listScenarioPacks(): ScenarioPack[] {
  return packs;
}

export function getScenarioPack(id: ScenarioId): ScenarioPack {
  const pack = packs.find((candidate) => candidate.id === id);
  if (!pack) {
    return packs[packs.length - 1];
  }
  return pack;
}
```

- [ ] **Step 5: Implement scene classifier**

Create `src/domain/classifyScene.ts`:

```ts
import { listScenarioPacks } from "./scenarioPacks";
import type { SceneCandidate } from "./types";

export function classifySceneFromText(rawText: string): SceneCandidate[] {
  const text = rawText.toLowerCase();
  const scored = listScenarioPacks().map((pack) => {
    const hits = pack.keywords.filter((keyword) => text.includes(keyword.toLowerCase())).length;
    const meterBonus = pack.id === "municipal_pipeline_survey" && /\d+\s*(米|m)\b/i.test(rawText) ? 2 : 0;
    const score = hits + meterBonus;
    const confidence = pack.id === "custom" ? 0.2 : Math.min(0.95, 0.25 + score * 0.12);

    return {
      id: pack.id,
      label: pack.label,
      confidence,
      reason: score > 0 ? `识别到 ${score} 个与“${pack.label}”相关的线索。` : `未识别到明确的“${pack.label}”线索。`
    };
  });

  return scored.sort((a, b) => b.confidence - a.confidence);
}
```

- [ ] **Step 6: Run scenario tests**

Run:

```bash
npm run test -- tests/domain/scenarioPacks.test.ts tests/domain/classifyScene.test.ts
```

Expected: PASS.

- [ ] **Step 7: Commit scenario logic**

```bash
git add src/domain/scenarioPacks.ts src/domain/classifyScene.ts tests/domain/scenarioPacks.test.ts tests/domain/classifyScene.test.ts
git commit -m "feat: add CAD scenario packs"
```

## Task 4: Generate Clarification Questions

**Files:**
- Create: `src/domain/questions.ts`
- Test: `tests/domain/questions.test.ts`

- [ ] **Step 1: Write failing question test**

Create `tests/domain/questions.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { sampleMunicipalProject } from "@/domain/sampleModel";
import { generateQuestions } from "@/domain/questions";

describe("generateQuestions", () => {
  it("asks high-priority global and entity-bound questions with custom and skip support", () => {
    const questions = generateQuestions(sampleMunicipalProject);

    expect(questions[0].prompt).toContain("单位");
    expect(questions[0].kind).toBe("single_choice");
    expect(questions[0].options.map((option) => option.label)).toContain("米");
    expect(questions[0].allowCustomAnswer).toBe(true);

    const baselineQuestion = questions.find((question) => question.targetIds.includes("entity_baseline_001"));
    expect(baselineQuestion?.prompt).toContain("基准线");
    expect(baselineQuestion?.allowSkip).toBe(true);
  });
});
```

- [ ] **Step 2: Verify the test fails**

Run:

```bash
npm run test -- tests/domain/questions.test.ts
```

Expected: FAIL because `generateQuestions` does not exist.

- [ ] **Step 3: Implement question generation**

Create `src/domain/questions.ts`:

```ts
import type { CadProject, ClarificationQuestion, QuestionOption } from "./types";

function option(id: string, label: string, value = id): QuestionOption {
  return { id, label, value };
}

export function generateQuestions(project: CadProject): ClarificationQuestion[] {
  const questions: ClarificationQuestion[] = [];

  questions.push({
    id: "question_global_unit",
    kind: "single_choice",
    priority: 10,
    prompt: "这张草图的主要尺寸单位是什么？",
    targetIds: [],
    options: [option("m", "米", "m"), option("cm", "厘米", "cm"), option("mm", "毫米", "mm")],
    allowCustomAnswer: true,
    allowSkip: false,
    unit: project.project.unit
  });

  const baseline = project.entities.find((entity) => entity.type === "baseline");
  if (baseline && baseline.confirmation !== "confirmed") {
    questions.push({
      id: `question_${baseline.id}_meaning`,
      kind: "single_choice",
      priority: 20,
      prompt: "图中的主长线/连续距离所在的线，应该作为哪一种基准线？",
      targetIds: [baseline.id],
      options: [
        option("road_edge", "道路边线"),
        option("road_center", "道路中心线"),
        option("pipeline_center", "管线中心线"),
        option("water_edge", "水沟/河道边界"),
        option("wall_boundary", "围墙或地界线")
      ],
      allowCustomAnswer: true,
      allowSkip: true
    });
  }

  for (const entity of project.entities) {
    if (entity.confirmation === "confirmed") {
      continue;
    }

    if (entity.type === "waterway") {
      questions.push({
        id: `question_${entity.id}_type`,
        kind: "single_choice",
        priority: 30,
        prompt: `“${entity.label}”这段波浪线表示什么？`,
        targetIds: [entity.id],
        options: [option("water_channel", "水沟"), option("river", "河道"), option("drainage_channel", "排水渠"), option("boundary", "边界线")],
        allowCustomAnswer: true,
        allowSkip: true
      });
    }

    if (entity.type === "building") {
      questions.push({
        id: `question_${entity.id}_size`,
        kind: "number",
        priority: 40,
        prompt: `“${entity.label}”旁边的尺寸是否表示建筑实际宽度？请输入确认后的宽度。`,
        targetIds: [entity.id],
        options: [],
        allowCustomAnswer: false,
        allowSkip: true,
        unit: project.project.unit
      });
    }
  }

  return questions.sort((a, b) => a.priority - b.priority);
}
```

- [ ] **Step 4: Run question tests**

Run:

```bash
npm run test -- tests/domain/questions.test.ts
```

Expected: PASS.

- [ ] **Step 5: Commit question generation**

```bash
git add src/domain/questions.ts tests/domain/questions.test.ts
git commit -m "feat: generate clarification questions"
```

## Task 5: Apply Answers And Validate Readiness

**Files:**
- Create: `src/domain/answers.ts`
- Create: `src/domain/validation.ts`
- Test: `tests/domain/answers.test.ts`
- Test: `tests/domain/validation.test.ts`

- [ ] **Step 1: Write failing answer test**

Create `tests/domain/answers.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { sampleMunicipalProject } from "@/domain/sampleModel";
import { applyAnswer } from "@/domain/answers";

describe("applyAnswer", () => {
  it("updates project unit from a global unit answer", () => {
    const updated = applyAnswer(sampleMunicipalProject, {
      id: "answer_001",
      questionId: "question_global_unit",
      value: "mm",
      skipped: false,
      createdAt: "2026-07-01T00:00:00.000Z"
    });

    expect(updated.project.unit).toBe("mm");
    expect(updated.answers).toHaveLength(1);
    expect(updated.versions.at(-1)?.reason).toBe("answered question_global_unit");
  });
});
```

- [ ] **Step 2: Write failing validation test**

Create `tests/domain/validation.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { sampleMunicipalProject } from "@/domain/sampleModel";
import { validateProject } from "@/domain/validation";

describe("validateProject", () => {
  it("reports unresolved objects before export", () => {
    const result = validateProject(sampleMunicipalProject);

    expect(result.readyForExport).toBe(false);
    expect(result.issues.some((issue) => issue.includes("entity_baseline_001"))).toBe(true);
  });
});
```

- [ ] **Step 3: Verify answer and validation tests fail**

Run:

```bash
npm run test -- tests/domain/answers.test.ts tests/domain/validation.test.ts
```

Expected: FAIL because the modules do not exist.

- [ ] **Step 4: Implement answer application**

Create `src/domain/answers.ts`:

```ts
import type { CadProject, CadUnit, ClarificationAnswer } from "./types";

function isCadUnit(value: string): value is CadUnit {
  return value === "m" || value === "cm" || value === "mm";
}

export function applyAnswer(project: CadProject, answer: ClarificationAnswer): CadProject {
  const next: CadProject = structuredClone(project);
  next.answers.push(answer);

  if (answer.questionId === "question_global_unit" && isCadUnit(answer.value)) {
    next.project.unit = answer.value;
    for (const constraint of next.constraints) {
      if (constraint.unit) {
        constraint.unit = answer.value;
      }
    }
  }

  if (answer.skipped) {
    const question = next.questions.find((item) => item.id === answer.questionId);
    for (const targetId of question?.targetIds ?? []) {
      const entity = next.entities.find((item) => item.id === targetId);
      if (entity) {
        entity.confirmation = "skipped";
      }
    }
  }

  next.versions.push({
    id: `version_${String(next.versions.length + 1).padStart(3, "0")}`,
    reason: `answered ${answer.questionId}`,
    createdAt: answer.createdAt
  });

  return next;
}
```

- [ ] **Step 5: Implement readiness validation**

Create `src/domain/validation.ts`:

```ts
import type { CadProject } from "./types";

export interface ValidationResult {
  readyForPreview: boolean;
  readyForExport: boolean;
  issues: string[];
}

export function validateProject(project: CadProject): ValidationResult {
  const issues: string[] = [];

  for (const entity of project.entities) {
    if (entity.confirmation !== "confirmed") {
      issues.push(`${entity.id} requires confirmation before final export.`);
    }
  }

  for (const constraint of project.constraints) {
    if (constraint.type === "distance" && (constraint.value === undefined || constraint.value <= 0)) {
      issues.push(`${constraint.id} has an invalid distance value.`);
    }
  }

  const hasDrawableGeometry = project.entities.some((entity) => entity.geometry.kind !== "text");

  return {
    readyForPreview: hasDrawableGeometry,
    readyForExport: hasDrawableGeometry && issues.length === 0,
    issues
  };
}
```

- [ ] **Step 6: Run answer and validation tests**

Run:

```bash
npm run test -- tests/domain/answers.test.ts tests/domain/validation.test.ts
```

Expected: PASS.

- [ ] **Step 7: Commit answer and validation logic**

```bash
git add src/domain/answers.ts src/domain/validation.ts tests/domain/answers.test.ts tests/domain/validation.test.ts
git commit -m "feat: apply clarification answers"
```

## Task 6: Build Preview Geometry

**Files:**
- Create: `src/domain/preview.ts`
- Test: `tests/domain/preview.test.ts`

- [ ] **Step 1: Write failing preview test**

Create `tests/domain/preview.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { sampleMunicipalProject } from "@/domain/sampleModel";
import { buildPreview } from "@/domain/preview";

describe("buildPreview", () => {
  it("converts CAD entities into SVG preview primitives", () => {
    const preview = buildPreview(sampleMunicipalProject);

    expect(preview.viewBox).toBe("0 0 900 420");
    expect(preview.items.some((item) => item.kind === "polyline")).toBe(true);
    expect(preview.items.some((item) => item.kind === "rect" && item.label === "邮政大厦")).toBe(true);
  });
});
```

- [ ] **Step 2: Verify the preview test fails**

Run:

```bash
npm run test -- tests/domain/preview.test.ts
```

Expected: FAIL because `buildPreview` does not exist.

- [ ] **Step 3: Implement preview conversion**

Create `src/domain/preview.ts`:

```ts
import type { CadProject } from "./types";

export type PreviewItem =
  | { kind: "polyline"; id: string; label: string; points: string; layer: string; confirmed: boolean }
  | { kind: "rect"; id: string; label: string; x: number; y: number; width: number; height: number; layer: string; confirmed: boolean }
  | { kind: "text"; id: string; label: string; x: number; y: number; value: string; layer: string; confirmed: boolean };

export interface PreviewModel {
  viewBox: string;
  items: PreviewItem[];
}

export function buildPreview(project: CadProject): PreviewModel {
  const items: PreviewItem[] = project.entities.map((entity) => {
    const confirmed = entity.confirmation === "confirmed";
    if (entity.geometry.kind === "line") {
      return {
        kind: "polyline",
        id: entity.id,
        label: entity.label,
        points: entity.geometry.points.map((point) => `${point.x},${point.y}`).join(" "),
        layer: entity.layer,
        confirmed
      };
    }

    if (entity.geometry.kind === "rectangle") {
      return {
        kind: "rect",
        id: entity.id,
        label: entity.label,
        x: entity.geometry.x,
        y: entity.geometry.y,
        width: entity.geometry.width,
        height: entity.geometry.height,
        layer: entity.layer,
        confirmed
      };
    }

    if (entity.geometry.kind === "text") {
      return {
        kind: "text",
        id: entity.id,
        label: entity.label,
        x: entity.geometry.x,
        y: entity.geometry.y,
        value: entity.geometry.value,
        layer: entity.layer,
        confirmed
      };
    }

    return {
      kind: "text",
      id: entity.id,
      label: entity.label,
      x: entity.geometry.x,
      y: entity.geometry.y,
      value: entity.label,
      layer: entity.layer,
      confirmed
    };
  });

  return {
    viewBox: "0 0 900 420",
    items
  };
}
```

- [ ] **Step 4: Run preview test**

Run:

```bash
npm run test -- tests/domain/preview.test.ts
```

Expected: PASS.

- [ ] **Step 5: Commit preview model**

```bash
git add src/domain/preview.ts tests/domain/preview.test.ts
git commit -m "feat: build vector preview model"
```

## Task 7: Export Editable DXF

**Files:**
- Create: `src/domain/dxf.ts`
- Test: `tests/domain/dxf.test.ts`

- [ ] **Step 1: Write failing DXF test**

Create `tests/domain/dxf.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { sampleMunicipalProject } from "@/domain/sampleModel";
import { exportDxf } from "@/domain/dxf";

describe("exportDxf", () => {
  it("exports lines, rectangles, text, and layers as ASCII DXF", () => {
    const dxf = exportDxf(sampleMunicipalProject);

    expect(dxf).toContain("SECTION");
    expect(dxf).toContain("LWPOLYLINE");
    expect(dxf).toContain("TEXT");
    expect(dxf).toContain("邮政大厦");
    expect(dxf).toContain("EOF");
  });
});
```

- [ ] **Step 2: Verify the DXF test fails**

Run:

```bash
npm run test -- tests/domain/dxf.test.ts
```

Expected: FAIL because `exportDxf` does not exist.

- [ ] **Step 3: Implement DXF export**

Create `src/domain/dxf.ts`:

```ts
import type { CadEntity, CadProject } from "./types";

function pair(code: number, value: string | number): string {
  return `${code}\n${value}`;
}

function layerName(entity: CadEntity): string {
  return entity.confirmation === "confirmed" ? entity.layer : "待确认";
}

function lineEntity(entity: CadEntity): string[] {
  if (entity.geometry.kind !== "line") {
    return [];
  }

  return [
    pair(0, "LWPOLYLINE"),
    pair(8, layerName(entity)),
    pair(90, entity.geometry.points.length),
    pair(70, 0),
    ...entity.geometry.points.flatMap((point) => [pair(10, point.x), pair(20, -point.y)])
  ];
}

function rectangleEntity(entity: CadEntity): string[] {
  if (entity.geometry.kind !== "rectangle") {
    return [];
  }

  const { x, y, width, height } = entity.geometry;
  const points = [
    { x, y },
    { x: x + width, y },
    { x: x + width, y: y + height },
    { x, y: y + height }
  ];

  return [
    pair(0, "LWPOLYLINE"),
    pair(8, layerName(entity)),
    pair(90, 4),
    pair(70, 1),
    ...points.flatMap((point) => [pair(10, point.x), pair(20, -point.y)])
  ];
}

function textEntity(entity: CadEntity): string[] {
  const geometry = entity.geometry;
  const x = geometry.kind === "text" || geometry.kind === "point" ? geometry.x : 0;
  const y = geometry.kind === "text" || geometry.kind === "point" ? geometry.y : 0;
  const value = geometry.kind === "text" ? geometry.value : entity.label;

  return [pair(0, "TEXT"), pair(8, layerName(entity)), pair(10, x), pair(20, -y), pair(40, 3.5), pair(1, value)];
}

export function exportDxf(project: CadProject): string {
  const body: string[] = [];

  for (const entity of project.entities) {
    if (entity.geometry.kind === "line") {
      body.push(...lineEntity(entity));
    } else if (entity.geometry.kind === "rectangle") {
      body.push(...rectangleEntity(entity));
      body.push(...textEntity(entity));
    } else {
      body.push(...textEntity(entity));
    }
  }

  return [
    pair(0, "SECTION"),
    pair(2, "HEADER"),
    pair(9, "$INSUNITS"),
    pair(70, project.project.unit === "m" ? 6 : project.project.unit === "cm" ? 5 : 4),
    pair(0, "ENDSEC"),
    pair(0, "SECTION"),
    pair(2, "ENTITIES"),
    ...body,
    pair(0, "ENDSEC"),
    pair(0, "EOF"),
    ""
  ].join("\n");
}
```

- [ ] **Step 4: Run DXF test**

Run:

```bash
npm run test -- tests/domain/dxf.test.ts
```

Expected: PASS.

- [ ] **Step 5: Commit DXF export**

```bash
git add src/domain/dxf.ts tests/domain/dxf.test.ts
git commit -m "feat: export editable DXF"
```

## Task 8: Add AI Analyzer Adapters

**Files:**
- Create: `src/ai/analyzer.ts`
- Create: `src/ai/openaiAnalyzer.ts`
- Test: `tests/ai/analyzer.test.ts`

- [ ] **Step 1: Write failing analyzer test**

Create `tests/ai/analyzer.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { analyzeSketchWithMock } from "@/ai/analyzer";

describe("analyzeSketchWithMock", () => {
  it("returns a municipal project with initial questions", async () => {
    const file = new File(["fake"], "field-sketch.jpg", { type: "image/jpeg" });
    const result = await analyzeSketchWithMock(file);

    expect(result.project.sceneCandidates[0].id).toBe("municipal_pipeline_survey");
    expect(result.questions.length).toBeGreaterThan(0);
  });
});
```

- [ ] **Step 2: Verify analyzer test fails**

Run:

```bash
npm run test -- tests/ai/analyzer.test.ts
```

Expected: FAIL because `@/ai/analyzer` does not exist.

- [ ] **Step 3: Implement mock analyzer**

Create `src/ai/analyzer.ts`:

```ts
import { classifySceneFromText } from "@/domain/classifyScene";
import { generateQuestions } from "@/domain/questions";
import { sampleMunicipalProject } from "@/domain/sampleModel";
import type { CadProject } from "@/domain/types";

export interface SketchAnalyzer {
  analyze(file: File): Promise<CadProject>;
}

export async function analyzeSketchWithMock(file: File): Promise<CadProject> {
  const project: CadProject = structuredClone(sampleMunicipalProject);
  project.project.id = `project_${Date.now()}`;
  project.project.sourceImageName = file.name;
  project.project.sceneCandidates = classifySceneFromText("邮政大厦 雨水检查井 50米 17米 水沟 124米");
  project.questions = generateQuestions(project);
  return project;
}
```

- [ ] **Step 4: Implement OpenAI analyzer**

Create `src/ai/openaiAnalyzer.ts`:

```ts
import OpenAI from "openai";
import { classifySceneFromText } from "@/domain/classifyScene";
import { generateQuestions } from "@/domain/questions";
import { sampleMunicipalProject } from "@/domain/sampleModel";
import type { CadProject } from "@/domain/types";

const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function fileToDataUrl(file: File): Promise<string> {
  const arrayBuffer = await file.arrayBuffer();
  const base64 = Buffer.from(arrayBuffer).toString("base64");
  return `data:${file.type || "image/jpeg"};base64,${base64}`;
}

export async function analyzeSketchWithOpenAI(file: File): Promise<CadProject> {
  if (!process.env.OPENAI_API_KEY) {
    throw new Error("OPENAI_API_KEY is required when AI_PROVIDER=openai.");
  }

  const imageUrl = await fileToDataUrl(file);
  const response = await client.responses.create({
    model: process.env.OPENAI_MODEL || "gpt-5.5",
    input: [
      {
        role: "user",
        content: [
          {
            type: "input_text",
            text: "请识别这张手绘 CAD 草图中的文字、尺寸、线条、建筑物、井、道路、水沟和不确定对象。只返回简洁中文摘要。"
          },
          {
            type: "input_image",
            image_url: imageUrl
          }
        ]
      }
    ]
  });

  const extractedText = response.output_text || "";
  const project: CadProject = structuredClone(sampleMunicipalProject);
  project.project.id = `project_${Date.now()}`;
  project.project.sourceImageName = file.name;
  project.project.sceneCandidates = classifySceneFromText(extractedText);
  project.questions = generateQuestions(project);
  project.versions.push({
    id: `version_${String(project.versions.length + 1).padStart(3, "0")}`,
    reason: "openai sketch analysis",
    createdAt: new Date().toISOString()
  });
  return project;
}
```

- [ ] **Step 5: Run analyzer tests**

Run:

```bash
npm run test -- tests/ai/analyzer.test.ts
```

Expected: PASS.

- [ ] **Step 6: Commit analyzer adapters**

```bash
git add src/ai/analyzer.ts src/ai/openaiAnalyzer.ts tests/ai/analyzer.test.ts
git commit -m "feat: add sketch analyzer adapters"
```

## Task 9: Add API Routes And Project Store

**Files:**
- Create: `src/server/projects.ts`
- Create: `src/app/api/analyze/route.ts`
- Create: `src/app/api/projects/[id]/scene/route.ts`
- Create: `src/app/api/projects/[id]/answer/route.ts`
- Create: `src/app/api/projects/[id]/export/route.ts`
- Test: `tests/server/projects.test.ts`

- [ ] **Step 1: Write failing project store test**

Create `tests/server/projects.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { sampleMunicipalProject } from "@/domain/sampleModel";
import { getProject, saveProject } from "@/server/projects";

describe("project store", () => {
  it("saves and retrieves a project by id", () => {
    saveProject(sampleMunicipalProject);

    expect(getProject(sampleMunicipalProject.project.id)?.project.id).toBe(sampleMunicipalProject.project.id);
  });
});
```

- [ ] **Step 2: Verify project store test fails**

Run:

```bash
npm run test -- tests/server/projects.test.ts
```

Expected: FAIL because `@/server/projects` does not exist.

- [ ] **Step 3: Implement in-memory project store**

Create `src/server/projects.ts`:

```ts
import type { CadProject } from "@/domain/types";

const projects = new Map<string, CadProject>();

export function saveProject(project: CadProject): CadProject {
  projects.set(project.project.id, project);
  return project;
}

export function getProject(projectId: string): CadProject | undefined {
  return projects.get(projectId);
}
```

- [ ] **Step 4: Add analyze route**

Create `src/app/api/analyze/route.ts`:

```ts
import { NextResponse } from "next/server";
import { analyzeSketchWithMock } from "@/ai/analyzer";
import { analyzeSketchWithOpenAI } from "@/ai/openaiAnalyzer";
import { saveProject } from "@/server/projects";

export async function POST(request: Request) {
  const formData = await request.formData();
  const file = formData.get("file");

  if (!(file instanceof File)) {
    return NextResponse.json({ error: "file is required" }, { status: 400 });
  }

  const provider = process.env.AI_PROVIDER || "mock";
  const project = provider === "openai" ? await analyzeSketchWithOpenAI(file) : await analyzeSketchWithMock(file);
  saveProject(project);

  return NextResponse.json(project);
}
```

- [ ] **Step 5: Add scene confirmation route**

Create `src/app/api/projects/[id]/scene/route.ts`:

```ts
import { NextResponse } from "next/server";
import { generateQuestions } from "@/domain/questions";
import { getProject, saveProject } from "@/server/projects";
import type { SceneCandidate, ScenarioId } from "@/domain/types";

function isScenarioId(value: string): value is ScenarioId {
  return ["municipal_pipeline_survey", "building_floor_plan", "cabinet_furniture", "mechanical_part", "electrical_wiring", "custom"].includes(value);
}

export async function POST(request: Request, context: { params: Promise<{ id: string }> }) {
  const { id } = await context.params;
  const project = getProject(id);

  if (!project) {
    return NextResponse.json({ error: "project not found" }, { status: 404 });
  }

  const body = (await request.json()) as { scene: string; customScene?: string };
  const scene: ScenarioId = isScenarioId(body.scene) ? body.scene : "custom";
  project.project.scene = scene;
  project.project.status = "clarifying";

  if (scene === "custom" && body.customScene) {
    const customCandidate: SceneCandidate = {
      id: "custom",
      label: body.customScene,
      confidence: 1,
      reason: "用户手动输入的场景。"
    };
    project.project.sceneCandidates = [customCandidate, ...project.project.sceneCandidates.filter((candidate) => candidate.id !== "custom")];
  }

  project.questions = generateQuestions(project);
  project.versions.push({
    id: `version_${String(project.versions.length + 1).padStart(3, "0")}`,
    reason: `confirmed scene ${scene}`,
    createdAt: new Date().toISOString()
  });

  saveProject(project);
  return NextResponse.json(project);
}
```

- [ ] **Step 6: Add answer route**

Create `src/app/api/projects/[id]/answer/route.ts`:

```ts
import { NextResponse } from "next/server";
import { applyAnswer } from "@/domain/answers";
import { generateQuestions } from "@/domain/questions";
import { validateProject } from "@/domain/validation";
import { getProject, saveProject } from "@/server/projects";
import type { ClarificationAnswer } from "@/domain/types";

export async function POST(request: Request, context: { params: Promise<{ id: string }> }) {
  const { id } = await context.params;
  const project = getProject(id);

  if (!project) {
    return NextResponse.json({ error: "project not found" }, { status: 404 });
  }

  const answer = (await request.json()) as ClarificationAnswer;
  const updated = applyAnswer(project, answer);
  updated.questions = generateQuestions(updated).filter((question) => !updated.answers.some((item) => item.questionId === question.id));
  updated.project.status = validateProject(updated).readyForExport ? "export_ready" : "clarifying";
  saveProject(updated);

  return NextResponse.json({
    project: updated,
    validation: validateProject(updated)
  });
}
```

- [ ] **Step 7: Add export route**

Create `src/app/api/projects/[id]/export/route.ts`:

```ts
import { NextResponse } from "next/server";
import { exportDxf } from "@/domain/dxf";
import { validateProject } from "@/domain/validation";
import { getProject } from "@/server/projects";

export async function GET(_request: Request, context: { params: Promise<{ id: string }> }) {
  const { id } = await context.params;
  const project = getProject(id);

  if (!project) {
    return NextResponse.json({ error: "project not found" }, { status: 404 });
  }

  const validation = validateProject(project);
  if (!validation.readyForPreview) {
    return NextResponse.json({ error: "project has no drawable geometry", validation }, { status: 422 });
  }

  return new Response(exportDxf(project), {
    status: 200,
    headers: {
      "Content-Type": "application/dxf",
      "Content-Disposition": `attachment; filename="${project.project.id}.dxf"`
    }
  });
}
```

- [ ] **Step 8: Run server tests**

Run:

```bash
npm run test -- tests/server/projects.test.ts
```

Expected: PASS.

- [ ] **Step 9: Commit API layer**

```bash
git add src/server/projects.ts src/app/api/analyze/route.ts src/app/api/projects/[id]/scene/route.ts src/app/api/projects/[id]/answer/route.ts src/app/api/projects/[id]/export/route.ts tests/server/projects.test.ts
git commit -m "feat: add project API routes"
```

## Task 10: Build Upload And Clarification UI

**Files:**
- Modify: `src/app/page.tsx`
- Create: `src/components/FileUpload.tsx`
- Create: `src/components/SceneConfirmation.tsx`
- Create: `src/components/QuestionPanel.tsx`
- Test: `tests/components/questionPanel.test.tsx`

- [ ] **Step 1: Write failing question panel test**

Create `tests/components/questionPanel.test.tsx`:

```tsx
import { fireEvent, render, screen } from "@testing-library/react";
import { describe, expect, it, vi } from "vitest";
import { QuestionPanel } from "@/components/QuestionPanel";
import type { ClarificationQuestion } from "@/domain/types";

const question: ClarificationQuestion = {
  id: "question_global_unit",
  kind: "single_choice",
  priority: 10,
  prompt: "这张草图的主要尺寸单位是什么？",
  targetIds: [],
  options: [
    { id: "m", label: "米", value: "m" },
    { id: "mm", label: "毫米", value: "mm" }
  ],
  allowCustomAnswer: true,
  allowSkip: false
};

describe("QuestionPanel", () => {
  it("submits a custom answer when the user chooses other input", () => {
    const onSubmit = vi.fn();
    render(<QuestionPanel question={question} onSubmit={onSubmit} />);

    fireEvent.change(screen.getByLabelText("其他，请输入"), { target: { value: "公里" } });
    fireEvent.click(screen.getByRole("button", { name: "提交回答" }));

    expect(onSubmit).toHaveBeenCalledWith(expect.objectContaining({ customValue: "公里" }));
  });
});
```

- [ ] **Step 2: Verify UI test fails**

Run:

```bash
npm run test -- tests/components/questionPanel.test.tsx
```

Expected: FAIL because `QuestionPanel` does not exist.

- [ ] **Step 3: Implement file upload component**

Create `src/components/FileUpload.tsx`:

```tsx
"use client";

export function FileUpload({ onFile }: { onFile: (file: File) => void }) {
  return (
    <label className="upload-box">
      <span>上传手绘草图</span>
      <input
        type="file"
        accept="image/*"
        onChange={(event) => {
          const file = event.target.files?.[0];
          if (file) {
            onFile(file);
          }
        }}
      />
    </label>
  );
}
```

- [ ] **Step 4: Implement scene confirmation component**

Create `src/components/SceneConfirmation.tsx`:

```tsx
"use client";

import type { SceneCandidate, ScenarioId } from "@/domain/types";

export function SceneConfirmation({
  candidates,
  onConfirm
}: {
  candidates: SceneCandidate[];
  onConfirm: (scene: ScenarioId | "custom", customValue?: string) => void;
}) {
  return (
    <section className="panel">
      <h2>确认图纸场景</h2>
      {candidates.map((candidate) => (
        <button key={candidate.id} type="button" onClick={() => onConfirm(candidate.id)}>
          {candidate.label} · {Math.round(candidate.confidence * 100)}%
        </button>
      ))}
      <form
        onSubmit={(event) => {
          event.preventDefault();
          const form = new FormData(event.currentTarget);
          onConfirm("custom", String(form.get("customScene") || ""));
        }}
      >
        <label>
          其他，请输入
          <input name="customScene" />
        </label>
        <button type="submit">确认自定义场景</button>
      </form>
    </section>
  );
}
```

- [ ] **Step 5: Implement question panel**

Create `src/components/QuestionPanel.tsx`:

```tsx
"use client";

import { useState } from "react";
import type { ClarificationAnswer, ClarificationQuestion } from "@/domain/types";

export function QuestionPanel({
  question,
  onSubmit
}: {
  question: ClarificationQuestion;
  onSubmit: (answer: Omit<ClarificationAnswer, "id" | "questionId" | "createdAt">) => void;
}) {
  const [selected, setSelected] = useState(question.options[0]?.value || "");
  const [customValue, setCustomValue] = useState("");
  const [numericValue, setNumericValue] = useState("");

  return (
    <section className="panel">
      <h2>澄清问题</h2>
      <p>{question.prompt}</p>
      {question.options.map((option) => (
        <label key={option.id} className="choice-row">
          <input type="radio" name={question.id} value={option.value} checked={selected === option.value} onChange={() => setSelected(option.value)} />
          {option.label}
        </label>
      ))}
      {question.kind === "number" && (
        <label>
          数值{question.unit ? `（${question.unit}）` : ""}
          <input aria-label="数值" inputMode="decimal" value={numericValue} onChange={(event) => setNumericValue(event.target.value)} />
        </label>
      )}
      {question.allowCustomAnswer && (
        <label>
          其他，请输入
          <input aria-label="其他，请输入" value={customValue} onChange={(event) => setCustomValue(event.target.value)} />
        </label>
      )}
      <div className="actions">
        <button
          type="button"
          onClick={() =>
            onSubmit({
              value: selected,
              customValue: customValue || undefined,
              numericValue: numericValue ? Number(numericValue) : undefined,
              unit: question.unit,
              skipped: false
            })
          }
        >
          提交回答
        </button>
        {question.allowSkip && (
          <button type="button" onClick={() => onSubmit({ value: "", skipped: true })}>
            暂时跳过
          </button>
        )}
      </div>
    </section>
  );
}
```

- [ ] **Step 6: Replace page with workflow state**

Modify `src/app/page.tsx`:

```tsx
"use client";

import { useMemo, useState } from "react";
import { FileUpload } from "@/components/FileUpload";
import { QuestionPanel } from "@/components/QuestionPanel";
import { SceneConfirmation } from "@/components/SceneConfirmation";
import type { CadProject, ClarificationAnswer, ScenarioId } from "@/domain/types";

export default function HomePage() {
  const [project, setProject] = useState<CadProject | null>(null);
  const [sceneConfirmed, setSceneConfirmed] = useState(false);
  const [loading, setLoading] = useState(false);
  const currentQuestion = useMemo(() => project?.questions[0], [project]);

  async function analyze(file: File) {
    setLoading(true);
    const formData = new FormData();
    formData.append("file", file);
    const response = await fetch("/api/analyze", { method: "POST", body: formData });
    setProject(await response.json());
    setLoading(false);
  }

  async function confirmScene(scene: ScenarioId | "custom", customScene?: string) {
    if (!project) {
      return;
    }

    const response = await fetch(`/api/projects/${project.project.id}/scene`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ scene, customScene })
    });
    setProject(await response.json());
    setSceneConfirmed(true);
  }

  async function submitAnswer(answer: Omit<ClarificationAnswer, "id" | "questionId" | "createdAt">) {
    if (!project || !currentQuestion) {
      return;
    }
    const response = await fetch(`/api/projects/${project.project.id}/answer`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        ...answer,
        id: `answer_${Date.now()}`,
        questionId: currentQuestion.id,
        createdAt: new Date().toISOString()
      })
    });
    const payload = await response.json();
    setProject(payload.project);
  }

  return (
    <main className="app-shell">
      <section className="workspace">
        <header className="masthead">
          <h1>CAD Helper</h1>
        </header>
        {!project && <FileUpload onFile={analyze} />}
        {loading && <p>正在分析草图...</p>}
        {project && !sceneConfirmed && <SceneConfirmation candidates={project.project.sceneCandidates} onConfirm={confirmScene} />}
        {project && sceneConfirmed && currentQuestion && <QuestionPanel question={currentQuestion} onSubmit={submitAnswer} />}
      </section>
    </main>
  );
}
```

- [ ] **Step 7: Run component test**

Run:

```bash
npm run test -- tests/components/questionPanel.test.tsx
```

Expected: PASS.

- [ ] **Step 8: Commit upload and question UI**

```bash
git add src/app/page.tsx src/components/FileUpload.tsx src/components/SceneConfirmation.tsx src/components/QuestionPanel.tsx tests/components/questionPanel.test.tsx
git commit -m "feat: add upload and clarification UI"
```

## Task 11: Add Preview, Inspector, And Export UI

**Files:**
- Modify: `src/app/page.tsx`
- Create: `src/components/PreviewCanvas.tsx`
- Create: `src/components/InspectorPanel.tsx`
- Create: `src/components/ExportPanel.tsx`
- Test: `tests/components/previewCanvas.test.tsx`

- [ ] **Step 1: Write failing preview component test**

Create `tests/components/previewCanvas.test.tsx`:

```tsx
import { render, screen } from "@testing-library/react";
import { describe, expect, it } from "vitest";
import { PreviewCanvas } from "@/components/PreviewCanvas";
import { sampleMunicipalProject } from "@/domain/sampleModel";

describe("PreviewCanvas", () => {
  it("renders building labels from the preview model", () => {
    render(<PreviewCanvas project={sampleMunicipalProject} />);

    expect(screen.getByText("邮政大厦")).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Verify preview component test fails**

Run:

```bash
npm run test -- tests/components/previewCanvas.test.tsx
```

Expected: FAIL because `PreviewCanvas` does not exist.

- [ ] **Step 3: Implement preview canvas**

Create `src/components/PreviewCanvas.tsx`:

```tsx
"use client";

import { buildPreview } from "@/domain/preview";
import type { CadProject } from "@/domain/types";

export function PreviewCanvas({ project }: { project: CadProject }) {
  const preview = buildPreview(project);

  return (
    <section className="preview-panel">
      <h2>矢量 demo</h2>
      <svg viewBox={preview.viewBox} role="img" aria-label="CAD vector preview">
        {preview.items.map((item) => {
          const className = item.confirmed ? "preview-confirmed" : "preview-pending";
          if (item.kind === "polyline") {
            return <polyline key={item.id} className={className} points={item.points} fill="none" />;
          }
          if (item.kind === "rect") {
            return (
              <g key={item.id}>
                <rect className={className} x={item.x} y={item.y} width={item.width} height={item.height} fill="none" />
                <text x={item.x + 8} y={item.y + 24}>
                  {item.label}
                </text>
              </g>
            );
          }
          return (
            <text key={item.id} x={item.x} y={item.y}>
              {item.value}
            </text>
          );
        })}
      </svg>
    </section>
  );
}
```

- [ ] **Step 4: Implement inspector panel**

Create `src/components/InspectorPanel.tsx`:

```tsx
"use client";

import { validateProject } from "@/domain/validation";
import type { CadProject } from "@/domain/types";

export function InspectorPanel({ project }: { project: CadProject }) {
  const validation = validateProject(project);

  return (
    <aside className="panel">
      <h2>对象与待确认项</h2>
      <ul>
        {project.entities.map((entity) => (
          <li key={entity.id}>
            {entity.label} · {entity.type} · {entity.confirmation}
          </li>
        ))}
      </ul>
      <h3>检查结果</h3>
      <ul>
        {validation.issues.map((issue) => (
          <li key={issue}>{issue}</li>
        ))}
      </ul>
    </aside>
  );
}
```

- [ ] **Step 5: Implement export panel**

Create `src/components/ExportPanel.tsx`:

```tsx
"use client";

import { validateProject } from "@/domain/validation";
import type { CadProject } from "@/domain/types";

export function ExportPanel({ project }: { project: CadProject }) {
  const validation = validateProject(project);
  const href = `/api/projects/${project.project.id}/export`;

  return (
    <section className="panel">
      <h2>导出 DXF</h2>
      <p>{validation.readyForExport ? "已满足最终导出条件。" : "仍有对象待确认，可先下载预览 DXF 或继续澄清。"}</p>
      <a className="button-link" href={href}>
        下载 DXF
      </a>
    </section>
  );
}
```

- [ ] **Step 6: Wire preview into page**

Modify `src/app/page.tsx` so the imports include:

```tsx
import { ExportPanel } from "@/components/ExportPanel";
import { InspectorPanel } from "@/components/InspectorPanel";
import { PreviewCanvas } from "@/components/PreviewCanvas";
```

Add this block below the question panel:

```tsx
{project && (
  <section className="work-grid">
    <PreviewCanvas project={project} />
    <div className="side-stack">
      <InspectorPanel project={project} />
      <ExportPanel project={project} />
    </div>
  </section>
)}
```

- [ ] **Step 7: Add preview styles**

Append to `src/styles/globals.css`:

```css
.masthead {
  margin-bottom: 20px;
}

.upload-box,
.panel,
.preview-panel {
  display: grid;
  gap: 12px;
  padding: 16px;
  border: 1px solid #d6dde6;
  border-radius: 8px;
  background: #ffffff;
}

.choice-row {
  display: flex;
  gap: 8px;
  align-items: center;
}

.actions {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.work-grid {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 360px;
  gap: 16px;
  margin-top: 16px;
}

.side-stack {
  display: grid;
  gap: 16px;
}

.preview-panel svg {
  width: 100%;
  aspect-ratio: 900 / 420;
  border: 1px solid #d6dde6;
  background: #fbfcfe;
}

.preview-confirmed {
  stroke: #1f7a4d;
  stroke-width: 2;
}

.preview-pending {
  stroke: #d97706;
  stroke-width: 2;
  stroke-dasharray: 8 5;
}

.button-link,
button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-height: 36px;
  padding: 0 12px;
  border: 1px solid #2b5c8a;
  border-radius: 6px;
  background: #ffffff;
  color: #163b5c;
  text-decoration: none;
  cursor: pointer;
}

@media (max-width: 900px) {
  .work-grid {
    grid-template-columns: 1fr;
  }
}
```

- [ ] **Step 8: Run preview component test**

Run:

```bash
npm run test -- tests/components/previewCanvas.test.tsx
```

Expected: PASS.

- [ ] **Step 9: Commit preview and export UI**

```bash
git add src/app/page.tsx src/components/PreviewCanvas.tsx src/components/InspectorPanel.tsx src/components/ExportPanel.tsx src/styles/globals.css tests/components/previewCanvas.test.tsx
git commit -m "feat: add preview and export UI"
```

## Task 12: Add End-To-End MVP Verification

**Files:**
- Create: `tests/e2e/mvp.spec.ts`
- Modify: `package.json`

- [ ] **Step 1: Install Playwright browser**

Run:

```bash
npx playwright install chromium
```

Expected: Chromium browser installation completes with code 0.

- [ ] **Step 2: Create E2E test**

Create `tests/e2e/mvp.spec.ts`:

```ts
import { expect, test } from "@playwright/test";

test("user can upload a sketch, answer a question, preview, and download DXF", async ({ page }) => {
  await page.goto("/");

  const fileInput = page.locator('input[type="file"]');
  await fileInput.setInputFiles({
    name: "field-sketch.jpg",
    mimeType: "image/jpeg",
    buffer: Buffer.from("fake image bytes")
  });

  await expect(page.getByText("确认图纸场景")).toBeVisible();
  await expect(page.getByText("澄清问题")).toBeVisible();
  await expect(page.getByText("邮政大厦")).toBeVisible();

  await page.getByLabel("其他，请输入").fill("米");
  await page.getByRole("button", { name: "提交回答" }).click();

  const download = page.waitForEvent("download");
  await page.getByRole("link", { name: "下载 DXF" }).click();
  const downloaded = await download;
  expect(downloaded.suggestedFilename()).toContain(".dxf");
});
```

- [ ] **Step 3: Run all unit tests**

Run:

```bash
npm run test
```

Expected: PASS for all unit and component tests.

- [ ] **Step 4: Run E2E test**

Run:

```bash
npm run e2e
```

Expected: PASS for Chromium.

- [ ] **Step 5: Run production build**

Run:

```bash
npm run build
```

Expected: build exits with code 0.

- [ ] **Step 6: Commit E2E verification**

```bash
git add tests/e2e/mvp.spec.ts package.json package-lock.json
git commit -m "test: cover sketch to DXF MVP flow"
```

## Final Verification

Run the full verification chain:

```bash
npm run test
npm run build
npm run e2e
git status -sb
```

Expected:

- Unit and component tests pass.
- Production build completes.
- E2E test downloads a `.dxf` file.
- `git status -sb` shows the branch with no uncommitted tracked changes.

## Acceptance Checklist

- [ ] A user can upload a sketch image.
- [ ] The app creates a structured `CadProject`.
- [ ] The app presents candidate scenes and keeps municipal survey as the first supported pack.
- [ ] Clarification questions support variable option counts.
- [ ] Clarification questions support “其他，请输入”.
- [ ] Numeric questions support numeric value capture and unit display.
- [ ] Answers are stored and written back into the project model.
- [ ] The preview renders lines, rectangles, text, pending states, and confirmed states.
- [ ] The inspector lists unresolved objects.
- [ ] DXF is generated from `CadProject`.
- [ ] DXF output includes editable line, polyline, rectangle, and text entities.
- [ ] OpenAI integration is isolated behind the analyzer adapter.
- [ ] The app works with `AI_PROVIDER=mock` without external API keys.
- [ ] The app can use `AI_PROVIDER=openai` when `OPENAI_API_KEY` and `OPENAI_MODEL` are set.

## Execution Handoff

Plan complete and saved to `docs/superpowers/plans/2026-07-01-ai-sketch-to-cad-mvp.md`. Two execution options:

1. **Subagent-Driven (recommended)** - Dispatch a fresh subagent per task, review between tasks, fast iteration.
2. **Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints.

Choose one before implementation starts.
