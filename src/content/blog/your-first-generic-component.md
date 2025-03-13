---
title: "Building your first generic Angular component"
description: "This article helps you to understand how to build your generic components on Angular"
pubDate: "Mar 13 2025"
heroImage: "/blog-placeholder-3.jpg"
---

> **TL;DR:** Building generic components in Angular requires careful consideration of trade-offs. This guide covers when to build them, how to design them effectively, and best practices for implementation, testing, and maintenance.

## Prerequisites

To get the most out of this article, you should have:

- Basic understanding of Angular components and services
- Familiarity with TypeScript generics
- Experience building simple Angular applications

---

## 1. Introduction

Building generic, reusable components in Angular can be both rewarding and challenging. While they promise to reduce duplication and standardize interfaces across your application, they also come with significant complexity and maintenance considerations. This guide will help you decide whether creating a generic component is right for your situation, and if so, how to approach it effectively. We'll explore best practices, common pitfalls, and technical considerations to ensure your component serves its users well.

### Who this article is for

- Angular developers looking to build reusable component libraries
- Team leads considering standardizing UI components across projects
- Developers who find themselves duplicating similar components across an application
- Anyone interested in modern Angular component architecture

---

## 2. Should You Build a Generic Component?

Before diving into implementation, consider these reasons why you might want to reconsider:

- **Resource constraints**: It's time consuming and maybe you don't have enough capacity for this
- **Existing solutions**: There are a lot of open source analogues that can be used; don't need to reinvent the wheel again. I suggest you make a proper research. It could save you a lot of effort.
- **Responsibility creep**: Generic components often try to cover too many responsibilities at once. This happens when people blindly follow the DRY rule. It's better to split functionality into separate specific components. It's okay to repeat some code.

> 💡 **Tip:** Before starting, search for existing solutions on npm, GitHub, or popular UI libraries. You might find something that meets 80% of your needs with much less effort.

---

## 3. Planning and Management Considerations

If you decide to proceed with building a generic component, consider these organizational aspects:

- **Incremental development**: Try to move faster in small increments; it helps to get more feedback and avoid getting stuck.
- **Support capacity**: Discuss support capacity with your manager before committing to maintaining a shared component.
- **Feedback strategy**: Try not to involve too many people in discussions at the beginning, as it will slow you down. Instead, gather feedback after release from your users (other developers).
- **Documentation and training**: Be ready for repetitive questions from other developers. Plan for small training sessions - concepts that are obvious to you may not be to others who haven't worked extensively with the component.

---

## 4. Technical Implementation

### API Design and Type Safety

- **Clear public API**: Your public API should be extremely clear for your users. Stick to naming conventions for inputs and outputs. Better to rely on what Angular and popular libraries already provide.
- **Type definitions**: Cover your generic component with types and interfaces. It should be clear what type each input expects and what type each output event has.
- **Backward compatibility**: At some point after your component is in use, you need to preserve the existing contract. Changes should happen only inside your component while the contract remains the same. You can ignore this during the MVP phase, but if you need to apply breaking changes, consider migration scripts or schematics.

### Modern Angular Features

- **Signals**: Utilize the latest stable Angular features like Signals, especially computed signals inside your component. Angular signals provide simple reactivity in your template. With computed signals, you can transform input data without contract changes.
- **Content projection**: This is an underestimated feature. Allow your users to choose what to project inside your component, making it more flexible to changes.

<details>
<summary>A generic data table component example</summary>

```typescript
// First, define our interfaces
export interface TableColumn<T = any> {
  key: keyof T;
  header: string;
  cellTemplate?: TemplateRef<unknown>;
  sortable?: boolean;
  width?: string;
}

export type TableSorting = "asc" | "desc";

// data-table.component.ts
import {
  Component,
  input,
  output,
  model,
  computed,
  contentChild,
  TemplateRef,
  ChangeDetectionStrategy,
} from "@angular/core";
import { NgTemplateOutlet } from "@angular/common";
import { GetCellValuePipe } from "./get-cell-value.pipe";
import { GetSortIconPipe } from "./get-sort-icon.pipe";

@Component({
  selector: "app-data-table",
  templateUrl: "./data-table.component.html",
  changeDetection: ChangeDetectionStrategy.OnPush,
  standalone: true,
  imports: [NgTemplateOutlet, GetCellValuePipe, GetSortIconPipe],
})
export class DataTableComponent<T = any> {
  // Events
  readonly rowClick = output<T>();

  // Input signals for reactive state management
  readonly data = input.required<T[]>();
  readonly columns = input.required<TableColumn<T>[]>();

  // Model signals for two-way binding
  readonly sortColumn = model<string | null>(null);
  readonly sortDirection = model<TableSorting>("asc");

  // Public API with computed signals
  readonly displayData = computed(() => {
    const data = [...this.data()];
    const sortCol = this.sortColumn();

    if (sortCol) {
      return data.sort((a: any, b: any) => {
        const aValue = a[sortCol];
        const bValue = b[sortCol];

        if (aValue === bValue) return 0;

        const comparison = aValue > bValue ? 1 : -1;
        return this.sortDirection() === "asc" ? comparison : -comparison;
      });
    }

    return data;
  });

  // Content projection for custom templates
  protected readonly headerTemplate =
    contentChild<TemplateRef<unknown>>("headerTemplate");
  protected readonly footerTemplate =
    contentChild<TemplateRef<unknown>>("footerTemplate");

  // Public methods
  protected sort(column: TableColumn<T>): void {
    if (!column.sortable) return;

    const key = column.key as string;

    if (this.sortColumn() === key) {
      // Toggle direction if already sorting by this column
      this.sortDirection.set(this.sortDirection() === "asc" ? "desc" : "asc");
    } else {
      // Set new sort column and reset direction
      this.sortColumn.set(key);
      this.sortDirection.set("asc");
    }
  }

  protected onRowClick(item: T): void {
    this.rowClick.emit(item);
  }
}
```

```html
<!-- data-table.component.html -->
<div class="table-container">
  <!-- Custom header template or default header -->
  @if (headerTemplate(); as headerTemplate) {
  <ng-container [ngTemplateOutlet]="headerTemplate"></ng-container>
  } @else {
  <div class="table-header">
    <h3>Data Table</h3>
  </div>
  }

  <!-- Table -->
  <table>
    <thead>
      <tr>
        @for (column of columns(); track column.key) {
        <th
          [style.width]="column.width"
          [class.sortable]="column.sortable"
          (click)="sort(column)"
        >
          {{ column.header }} @if (column.sortable) {
          <span class="sort-icon"
            >{{ column | getSortIcon : sortColumn() : sortDirection() }}</span
          >
          }
        </th>
        }
      </tr>
    </thead>
    <tbody>
      @for (item of displayData(); track item) {
      <tr (click)="onRowClick(item)">
        @for (column of columns(); track column.key) {
        <td>
          <!-- Custom cell template or default cell -->
          @if (column.cellTemplate) {
          <ng-container
            [ngTemplateOutlet]="column.cellTemplate"
            [ngTemplateOutletContext]="{ $implicit: item, column: column }"
          >
          </ng-container>
          } @else { {{ item | getCellValue : column.key }} }
        </td>
        }
      </tr>
      }
    </tbody>
  </table>

  <!-- Custom footer template -->
  @if (footerTemplate(); as footerTemplate) {
  <ng-container [ngTemplateOutlet]="footerTemplate"></ng-container>
  }
</div>
```

### Usage Example

```typescript
// users-page.component.ts
import {
  ChangeDetectionStrategy,
  Component,
  computed,
  Signal,
  TemplateRef,
  viewChild,
} from "@angular/core";
import {
  DataTableComponent,
  TableColumn,
} from "../data-table/data-table.component";

interface User {
  id: number;
  name: string;
  email: string;
  role: string;
  active: boolean;
}

@Component({
  selector: "app-users-page",
  templateUrl: "./users-page.component.html",
  styleUrl: "./users-page.component.css",
  changeDetection: ChangeDetectionStrategy.OnPush,
  standalone: true,
  imports: [DataTableComponent],
})
export class UsersPageComponent {
  protected readonly activeTemplate =
    viewChild<TemplateRef<unknown>>("activeTemplate");

  users: User[] = [
    {
      id: 1,
      name: "John Doe",
      email: "john@example.com",
      role: "Admin",
      active: true,
    },
    {
      id: 2,
      name: "Jane Smith",
      email: "jane@example.com",
      role: "User",
      active: false,
    },
    // More users...
  ];

  columns: Signal<TableColumn<User>[]> = computed(() => {
    const activeTemplate = this.activeTemplate();
    return [
      { key: "name", header: "Name", sortable: true },
      { key: "email", header: "Email", sortable: true },
      { key: "role", header: "Role", sortable: true },
      {
        key: "active",
        header: "Status",
        sortable: true,
        cellTemplate: activeTemplate,
      },
    ];
  });

  onUserClick(user: User): void {
    console.log("User clicked:", user);
    // Handle user selection
  }

  addUser(): void {
    // Handle adding a new user
  }
}
```

```html
<!-- users-page.component.html -->
<app-data-table
  [data]="users"
  [columns]="columns()"
  (rowClick)="onUserClick($event)"
>
  <ng-template #headerTemplate>
    <div class="custom-header">
      <h2>User Management</h2>
      <button (click)="addUser()">Add User</button>
    </div>
  </ng-template>
</app-data-table>
<ng-template #activeTemplate let-user>
  <span class="status-indicator" [class.active]="user.active">
    {{ user.active ? 'Active' : 'Inactive' }}
  </span>
</ng-template>
```

This example demonstrates several modern Angular features:

1. **Input/Output Signals**: Using `input()` and `output()` functions instead of decorators for reactive inputs and outputs
2. **Model Signals**: Using `model()` for two-way binding with the sortColumn and sortDirection
3. **New Control Flow Syntax**: Using `@if`, `@else`, and `@for` instead of structural directives like `*ngIf` and `*ngFor`
4. **Content Projection**: Using `contentChild` and `ng-template` for flexible content customization
5. **OnPush Change Detection**: For optimal performance
6. **Computed Signals**: For derived state like sorted data

These features make the component more reactive, type-safe, and maintainable while improving performance.

</details>

### Performance and Optimization

- **Change detection**: Change detection strategy should be OnPush. This is non-negotiable for reusable components.
- **Large datasets**: Consider optimizations if your component will be used with large data sets. For tables, implement virtual scrolling (can be configurable via input).
- **Memoization**: Use memoization techniques for expensive operations. For simpler cases, computed signals can handle this for you.
- **State management**: Don't abuse state as it will create difficulties in controlling your component. If you need state providers, separate them from other parts of your app. The same applies to localStorage or other storage - use separate keys to avoid conflicts.

> ⚠️ **Important:** Change detection strategy should be OnPush for all generic components. This is non-negotiable for performance reasons.

### Testing and Quality Assurance

- **Unit tests**: Cover your code with tests - at minimum, unit tests should verify how your component reacts to all inputs and confirm it emits all expected events/outputs.
- **E2E tests**: If you have capacity, write basic E2E tests for main scenarios. For a table component, cover pagination, sorting, and search actions to ensure all core functionality works properly after changes.
- **Cross-browser compatibility**: Try to cover all main browsers and devices if you have capacity. The "we only use Chrome" approach rarely lasts long.
- **Fallbacks**: If you have browser/device support limitations, discuss this with management. Either mark certain devices/browsers as not well supported, or disable component usage in those environments (though this is not ideal).

> 🧪 **Testing tip:** Create a comprehensive test suite early. It will save you time when refactoring and give you confidence when making changes.

### User Experience Considerations

- **Internationalization (i18n)**: Consider what types of users will interact with your component. Plan for date/number formatting, RTL support, and text translation.
- **Styling and theming**: Make styling configurable. CSS custom properties are one of the best solutions to let developers change appearance aspects.
- **Accessibility**: Accessibility should never be an afterthought. Ensure your component follows WCAG guidelines by implementing proper keyboard navigation (tab order, focus management), adding appropriate ARIA attributes (roles, labels, descriptions), and maintaining sufficient color contrast. Test with screen readers like NVDA or VoiceOver. Remember that accessibility benefits all users, not just those with disabilities. For interactive components like tables, dropdowns, or modals, implement proper keyboard controls (arrow keys, Escape key) and ensure focus trapping when necessary. Document accessibility features so developers using your component understand how to maintain them.

---

## 5. Documentation and Sharing

- **Demo page**: Create a demo page with configurable options, or a playground page hidden behind a dev flag that you can share with other developers.
- **Change communication**: Learn how to publish changes clearly so users are informed. Write changelogs, update your demo page/playground, and provide code examples.

> 📝 **Documentation tip:** Include both API documentation and usage examples. Show common patterns and edge cases to help developers understand how to use your component effectively.

---

## 6. Common Pitfalls to Avoid

- **Over-generalization**: Making components too generic often leads to complexity and maintenance issues. Focus on solving specific problems rather than creating a component that tries to do everything.

- **Poor documentation**: Generic components without clear documentation become unusable. Document not just the API, but also usage patterns, edge cases, and limitations.

- **Ignoring performance**: Generic components that don't consider performance can cause application-wide slowdowns. Always use OnPush change detection and consider the impact of your component on the overall application performance.

- **Tight coupling**: Avoid dependencies on specific services or state management solutions. Your component should work regardless of the application architecture it's used in.

---

## 7. Conclusion

Building a generic component requires careful planning, solid technical implementation, and ongoing maintenance. When done right, it can significantly improve development efficiency and application consistency. However, always weigh the benefits against the costs before embarking on this journey.

### Next Steps

- Start small with a simple generic component that solves a specific problem
- Join Angular communities to share your components and get feedback
- Consider contributing to open-source component libraries to learn from others

### Further Reading

- [Angular Documentation on Components](https://angular.dev/guide/components)
- [TypeScript Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [Angular Content Projection Guide](https://angular.dev/guide/components/content-projection)
- [Building Accessible Components](https://angular.dev/best-practices/a11y)

---

*This article was last updated on March 13, 2025. The code examples use Angular 19.*
