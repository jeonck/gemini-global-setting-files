## Gemini Added Memories
- When connecting a project to GitHub, the user prefers that I create the remote repository on GitHub first before initializing the local git repository and pushing to it.
- Do not ask for confirmation before pushing changes. Proceed with the push directly.
- The `react-feather` library can cause application crashes when icons are used directly in component definitions. Prefer using a different icon library or a more robust way of importing the icons.
- When using Vite with a subdirectory, ensure the `base` property in `vite.config.ts` and the `basename` prop on the React Router component are both set correctly.
- Pay close attention to syntax when performing `replace` operations, as even small typos like stray characters can break the build.
- When using the `marked` library in `MarkdownRenderer.tsx`, ensure that the `href` parameter in the custom link renderer is explicitly checked to be a string (`typeof href === 'string'`) before calling string methods like `startsWith`. This prevents `TypeError: href.startsWith is not a function` which can occur if `marked` provides a non-string value for `href` in certain scenarios.
- When using Vite with a subdirectory, ensure the `base` property in `vite.config.ts` and the `basename` prop on the React Router component are both set correctly, ideally using environment variables for flexible configuration between local development and deployment.
