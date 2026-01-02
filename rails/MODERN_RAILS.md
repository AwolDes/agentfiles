# Modern Rails with Hotwire: Decision Framework

## Core Philosophy: Progressive Enhancement

Always follow this hierarchy (simplest → most complex):
1. HTML & CSS (server-side rendering)
2. Turbo Drive (automatic, already enabled in Rails 7)
3. Turbo Frames (isolated page regions)
4. Turbo Streams (multiple updates or real-time)
5. Stimulus (JS glue and small interactions)
6. Other JS libraries (only when necessary)

**Rule: Start simple. Only move up the stack when you actually need the complexity.**

## Turbo 8 Page Refreshes & Morphing

### When to Use Page Refreshes
- **Content pages**: Pages where users expect to see "something new" (articles, timelines, dashboards, calendar views)
- **Parent-child relationships**: When editing a child resource redirects back to the parent view
- **Real-time collaboration**: Multiple users editing the same content simultaneously

### Implementation Pattern
```ruby
# Model: Enable broadcasts with one line
class Conversation < ApplicationRecord
  broadcasts_refreshes
end

# View: Subscribe to updates with one line
<%= turbo_stream_from @conversation %>

# Layout: Enable morphing globally
<%= turbo_refresh_method_tag :morph %>
<%= turbo_refresh_scroll_tag :preserve %>
```

### When Page Refreshes Trigger
1. **You're redirected to the same path you're already on** (e.g., `/calendar/weeks/33` → PATCH `/events/142` → redirect to `/calendar/weeks/33`)
2. **Someone else changes data** and the server broadcasts "refresh yourself" to other connected clients

### Design Pattern: Content Pages vs Sub-Pages
- **Content page**: The canonical view (e.g., calendar view, kanban board, post timeline)
- **Sub-elements**: Items edited *within* that view (events, cards, comments)
- **Redirect pattern**: Edit a sub-element → redirect back to content page
- **Scroll rule of thumb**: If users expect scroll to reset, it's a content page

### Trade-offs
- **Pro**: Dramatically less code than manual Turbo Streams
- **Pro**: Automatic real-time updates for all users
- **Con**: Slightly slower than targeted Turbo Streams (100-300ms)
- **Con**: More web server traffic (clients request full page)
- **Verdict**: Use Page Refreshes for most cases; only use manual Turbo Streams for high-performance needs like chat

## Turbo Frames

### When to Use
- Isolated, focused regions of a page with their own interactivity
- Loading/editing content inline without full page refresh
- Examples: "edit comment" button, dashboard widgets, trending items sidebar

### Best Practices
```erb
<%# Lazy-load frames when they become visible %>
<%= turbo_frame_tag "trending_items", src: trending_items_path, loading: "lazy" %>

<%# Target other frames on the page %>
<%= link_to "Edit", edit_comment_path(@comment), 
    data: { turbo_frame: "comment_form_area" } %>
```

- Extract frames into their own resourceful controllers
- Reuse existing partials where possible
- Keep the page functional with JS disabled

## Turbo Streams

### When to Use
- Need to update **multiple separate sections** of the page at once
- Async updates over WebSockets (real-time notifications, chat messages, "user online" indicators)
- When Turbo Frames aren't enough

### Embedding Streams in Frames
```erb
<%# You can embed turbo_stream in turbo_frame responses %>
<%= turbo_frame_tag 'show_package' do %>
  <%= render @package %>
  
  <%= turbo_stream.update 'current_message' do %>
    Currently viewing package #<%= @package.id %>
  <% end %>
<% end %>
```

### Multiple Streams in One Response
```erb
<%# app/views/packages/show.turbo_stream.erb %>
<%= turbo_stream.update 'show_package' do %>
  <%= render @package %>
<% end %>

<%= turbo_stream.update 'current_message' do %>
  Currently viewing package #<%= @package.id %>
<% end %>
```

### ActionCable for Real-Time
```erb
<%# Subscribe to a global stream %>
<%= turbo_stream_from 'world_messages' %>

<%# Subscribe to a scoped stream %>
<%= turbo_stream_from @conversation %>
```

## Stimulus

### When to Use
1. **Integrate other JS libraries** (Popper, tooltips, file uploaders)
2. **Small DOM updates** that don't warrant server round-trips
3. **User input management** (character counters, auto-submit, live calculations)
4. **Glue between Turbo parts** (refresh a frame on timer, submit form on change)

### Integration Pattern
```javascript
// Bind JS libraries that need to work with Turbo-injected elements
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ 'tooltip' ]

  tooltipTargetConnected(element) {
    new bootstrap.Tooltip(element)
  }
}
```

### Anti-Patterns (DON'T DO THIS)
- ❌ Writing templating logic in Stimulus (use Turbo Frames instead)
- ❌ Building large, page-specific, non-reusable controllers
- ❌ Injecting large HTML chunks via JS (use Turbo Streams)
- ❌ Doing validation only in Stimulus (always validate server-side too)
- ❌ Building a "hall-monitor" controller that manages the whole page

### Keep Controllers Small & Reusable
- Controllers should be focused and portable
- Aim for reusability across your app
- If it's getting complex, consider Turbo instead

## Decision Tree
```
Need interactivity?
├─ No → Plain HTML/CSS
└─ Yes
   ├─ Is it a content page with parent-child editing?
   │  └─ Yes → Page Refreshes (broadcasts_refreshes)
   │
   ├─ Is it one isolated region?
   │  └─ Yes → Turbo Frame
   │
   ├─ Need to update multiple regions at once?
   │  └─ Yes → Turbo Streams
   │
   ├─ Need real-time updates from other users/events?
   │  └─ Yes → Turbo Streams + ActionCable
   │
   ├─ Need to integrate a JS library or small DOM updates?
   │  └─ Yes → Stimulus
   │
   └─ Everything else
      └─ Reconsider if you need it at all
```

## Key Principles

1. **Default to server-side rendering** - Turbo Drive makes it fast
2. **Page Refreshes for content pages** - Minimal code, maximum real-time magic
3. **Frames for isolation** - Independent page regions
4. **Streams for coordination** - Multiple updates or WebSocket broadcasts
5. **Stimulus as glue** - Connect JS libraries and handle micro-interactions
6. **Always validate server-side** - Never trust client-side only
7. **Keep it simple** - Don't use complex solutions for simple problems

## Common Patterns

### Timeline/Feed Interface
```ruby
# Use Page Refreshes
class Post < ApplicationRecord
  broadcasts_refreshes
end
```

### Edit-in-Place
```erb
<%# Use Turbo Frame %>
<%= turbo_frame_tag dom_id(comment) do %>
  <%= render comment %>
<% end %>
```

### Chat Messages
```erb
<%# Use Turbo Streams + ActionCable %>
<%= turbo_stream_from @chat_room %>
```

### Character Counter
```javascript
// Use Stimulus
export default class extends Controller {
  static targets = [ 'input', 'counter' ]
  
  update() {
    this.counterTarget.textContent = this.inputTarget.value.length
  }
}
```