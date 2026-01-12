# UX Design Examples

## Empty State Examples

### Dashboard - No Projects

```tsx
<div className="flex flex-col items-center justify-center py-16 px-4 text-center">
  <div className="w-24 h-24 mb-6 rounded-full bg-gradient-to-br from-blue-100 to-purple-100 flex items-center justify-center">
    <FolderIcon className="w-12 h-12 text-blue-500" />
  </div>

  <h2 className="text-xl font-semibold text-gray-900 mb-2">
    No projects yet
  </h2>

  <p className="text-gray-500 mb-6 max-w-sm">
    This is where your creative work lives. Start by uploading an image
    to explore what's possible.
  </p>

  <Button size="lg">
    <PlusIcon className="w-5 h-5 mr-2" />
    Create Your First Project
  </Button>

  <p className="text-sm text-gray-400 mt-4">
    You have 2 free credits to get started
  </p>
</div>
```

### Search - No Results

```tsx
<div className="flex flex-col items-center py-12 text-center">
  <SearchIcon className="w-12 h-12 text-gray-300 mb-4" />

  <h3 className="text-lg font-medium text-gray-900 mb-1">
    No results for "{query}"
  </h3>

  <p className="text-gray-500 mb-4">
    Try different keywords or check your spelling
  </p>

  <div className="flex gap-3">
    <Button variant="outline" onClick={clearSearch}>
      Clear Search
    </Button>
    <Button onClick={showAll}>
      Browse All
    </Button>
  </div>
</div>
```

### Inbox - All Complete

```tsx
<div className="flex flex-col items-center py-16 text-center">
  <div className="w-20 h-20 mb-6 rounded-full bg-green-100 flex items-center justify-center">
    <CheckCircleIcon className="w-10 h-10 text-green-500" />
  </div>

  <h2 className="text-xl font-semibold text-gray-900 mb-2">
    All caught up!
  </h2>

  <p className="text-gray-500 mb-6">
    You've completed everything. Time for a break.
  </p>
</div>
```

---

## Onboarding Checklist Example

```tsx
const OnboardingChecklist = ({ tasks }) => {
  const completed = tasks.filter(t => t.done).length;
  const progress = (completed / tasks.length) * 100;

  return (
    <div className="bg-white rounded-xl shadow-sm border p-6 max-w-sm">
      <div className="flex items-center justify-between mb-4">
        <h3 className="font-semibold">Get Started</h3>
        <span className="text-sm text-gray-500">
          {completed}/{tasks.length}
        </span>
      </div>

      {/* Progress bar */}
      <div className="h-2 bg-gray-100 rounded-full mb-6 overflow-hidden">
        <div
          className="h-full bg-blue-500 rounded-full transition-all duration-500"
          style={{ width: `${progress}%` }}
        />
      </div>

      {/* Tasks */}
      <ul className="space-y-3">
        {tasks.map(task => (
          <li
            key={task.id}
            className={cn(
              "flex items-center gap-3 p-3 rounded-lg transition-colors",
              task.done ? "bg-gray-50" : "bg-blue-50 cursor-pointer hover:bg-blue-100"
            )}
            onClick={() => !task.done && task.action()}
          >
            <div className={cn(
              "w-6 h-6 rounded-full flex items-center justify-center",
              task.done ? "bg-green-500" : "bg-white border-2 border-blue-300"
            )}>
              {task.done && <CheckIcon className="w-4 h-4 text-white" />}
            </div>
            <span className={task.done ? "text-gray-400 line-through" : "text-gray-700"}>
              {task.label}
            </span>
            {!task.done && (
              <ChevronRightIcon className="w-5 h-5 text-gray-400 ml-auto" />
            )}
          </li>
        ))}
      </ul>
    </div>
  );
};

// Usage
<OnboardingChecklist tasks={[
  { id: 1, label: "Upload your first image", done: true },
  { id: 2, label: "Generate a variation", done: false, action: () => {} },
  { id: 3, label: "Save to your project", done: false, action: () => {} },
  { id: 4, label: "Download your creation", done: false, action: () => {} },
]} />
```

---

## Progressive Disclosure Example

### Settings with Expandable Sections

```tsx
const SettingsSection = ({ title, description, children, defaultOpen = false }) => {
  const [isOpen, setIsOpen] = useState(defaultOpen);

  return (
    <div className="border-b last:border-0">
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="w-full flex items-center justify-between py-4 text-left"
      >
        <div>
          <h3 className="font-medium text-gray-900">{title}</h3>
          <p className="text-sm text-gray-500">{description}</p>
        </div>
        <ChevronDownIcon
          className={cn(
            "w-5 h-5 text-gray-400 transition-transform",
            isOpen && "rotate-180"
          )}
        />
      </button>

      {isOpen && (
        <div className="pb-4 pl-4">
          {children}
        </div>
      )}
    </div>
  );
};
```

---

## CTA Button Examples

### Primary CTA with Loading State

```tsx
const PrimaryCTA = ({ loading, onClick, children }) => (
  <button
    onClick={onClick}
    disabled={loading}
    className={cn(
      "relative px-6 py-3 rounded-lg font-medium text-white",
      "bg-blue-600 hover:bg-blue-700 active:bg-blue-800",
      "transition-all duration-150",
      "disabled:opacity-70 disabled:cursor-not-allowed",
      "focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
    )}
  >
    {loading ? (
      <>
        <span className="opacity-0">{children}</span>
        <div className="absolute inset-0 flex items-center justify-center">
          <Spinner className="w-5 h-5" />
        </div>
      </>
    ) : (
      <span className="flex items-center gap-2">
        {children}
        <ArrowRightIcon className="w-4 h-4" />
      </span>
    )}
  </button>
);

// Usage
<PrimaryCTA loading={isSubmitting} onClick={handleSubmit}>
  Start Creating
</PrimaryCTA>
```

### CTA with Success State

```tsx
const [state, setState] = useState<'idle' | 'loading' | 'success'>('idle');

<button
  onClick={async () => {
    setState('loading');
    await action();
    setState('success');
    setTimeout(() => setState('idle'), 2000);
  }}
  className="..."
>
  {state === 'idle' && 'Save Changes'}
  {state === 'loading' && <Spinner />}
  {state === 'success' && (
    <span className="flex items-center gap-2 text-green-600">
      <CheckIcon className="w-5 h-5" />
      Saved!
    </span>
  )}
</button>
```

---

## Micro-interaction Examples

### Inline Form Validation

```tsx
const Input = ({ label, error, success, ...props }) => (
  <div className="space-y-1">
    <label className="text-sm font-medium text-gray-700">
      {label}
    </label>
    <div className="relative">
      <input
        className={cn(
          "w-full px-4 py-2 border rounded-lg transition-colors",
          "focus:outline-none focus:ring-2",
          error && "border-red-300 focus:ring-red-500",
          success && "border-green-300 focus:ring-green-500",
          !error && !success && "border-gray-300 focus:ring-blue-500"
        )}
        {...props}
      />
      {success && (
        <CheckCircleIcon className="absolute right-3 top-1/2 -translate-y-1/2 w-5 h-5 text-green-500" />
      )}
      {error && (
        <ExclamationCircleIcon className="absolute right-3 top-1/2 -translate-y-1/2 w-5 h-5 text-red-500" />
      )}
    </div>
    {error && (
      <p className="text-sm text-red-600">{error}</p>
    )}
  </div>
);
```

### Toast Notification

```tsx
const Toast = ({ type, message, onClose }) => (
  <div
    className={cn(
      "flex items-center gap-3 px-4 py-3 rounded-lg shadow-lg",
      "animate-in slide-in-from-top-5 fade-in duration-300",
      type === 'success' && "bg-green-50 border border-green-200",
      type === 'error' && "bg-red-50 border border-red-200",
      type === 'info' && "bg-blue-50 border border-blue-200"
    )}
  >
    {type === 'success' && <CheckCircleIcon className="w-5 h-5 text-green-500" />}
    {type === 'error' && <XCircleIcon className="w-5 h-5 text-red-500" />}
    {type === 'info' && <InformationCircleIcon className="w-5 h-5 text-blue-500" />}

    <p className="text-sm text-gray-700">{message}</p>

    <button onClick={onClose} className="ml-auto p-1 hover:bg-gray-100 rounded">
      <XIcon className="w-4 h-4 text-gray-400" />
    </button>
  </div>
);
```

---

## Skeleton Loading Example

```tsx
const ProjectCardSkeleton = () => (
  <div className="bg-white rounded-xl border p-4 animate-pulse">
    {/* Image placeholder */}
    <div className="aspect-video bg-gray-200 rounded-lg mb-4" />

    {/* Title placeholder */}
    <div className="h-5 bg-gray-200 rounded w-3/4 mb-2" />

    {/* Description placeholder */}
    <div className="h-4 bg-gray-100 rounded w-full mb-1" />
    <div className="h-4 bg-gray-100 rounded w-2/3" />

    {/* Button placeholder */}
    <div className="h-10 bg-gray-200 rounded-lg mt-4" />
  </div>
);

// Usage - show skeletons while loading
{isLoading ? (
  <div className="grid grid-cols-3 gap-6">
    {[...Array(6)].map((_, i) => (
      <ProjectCardSkeleton key={i} />
    ))}
  </div>
) : (
  <div className="grid grid-cols-3 gap-6">
    {projects.map(project => (
      <ProjectCard key={project.id} project={project} />
    ))}
  </div>
)}
```

---

## Contextual Tooltip Example

```tsx
const FeatureTooltip = ({ feature, children }) => {
  const [hasSeenTip, setHasSeenTip] = useLocalStorage(`tip-${feature}`, false);
  const [isOpen, setIsOpen] = useState(!hasSeenTip);

  if (hasSeenTip) return children;

  return (
    <div className="relative">
      {children}

      {isOpen && (
        <div className="absolute top-full left-1/2 -translate-x-1/2 mt-2 z-50">
          <div className="bg-gray-900 text-white px-4 py-3 rounded-lg shadow-xl max-w-xs">
            <p className="text-sm mb-2">
              {TIPS[feature].message}
            </p>
            <button
              onClick={() => {
                setIsOpen(false);
                setHasSeenTip(true);
              }}
              className="text-xs text-blue-300 hover:text-blue-200"
            >
              Got it
            </button>
          </div>
          {/* Arrow */}
          <div className="absolute -top-1 left-1/2 -translate-x-1/2 w-2 h-2 bg-gray-900 rotate-45" />
        </div>
      )}
    </div>
  );
};
```
