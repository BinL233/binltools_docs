## Vim 
### Navigation
- `h` `j` `k` `l`: Left, Down, Up, Right
- `w`: Jump to the start of next word
- `b`: Jump to the start of previous word
- `0`: Jump to the start of the line
- `$`: Jump to the end of the line
- `gg`: Jump to the start of the file
- `G`: Jump to the end of the file
- `:<number>`: Jump to the specific line
- `C-u`: page up
- `C-d`: Page down

### Edit
- `i`: Insert before the cursor
- `a`: Insert after the cursor
- `o`: Open a new line
- `x`: Delete a single char
- `dd`: Delete the line (cut)
- `yy`: Yank the line (copy)
- `p`: Paste
- `u`: Undo
- `C+r`: Redo
- `ciw`: Change inner word (deletes the word you're on and puts you in Insert mode)
- `diw`: Delete inner word
- `dw`: Delete from cursor to the start of next word

### Search
- `\<pattern>`: Search in file
    - `n`: Go next
    - `N`: Go previous
- `:%s/old/new/g`: Replace old to new globally
- `:%s/old/new/gc`: Same as above but need confirmation

### Others
- `.`: Repeat the last command
- `sp`: Split window horizontally
- `vsp`: Split window vertically

## Nvim & Plugins
### Edit
- `gc`: Comment line/range

### LSP
- `[d`: Jump to previous error
- `]d`: Jump to next error
- `<C-w>d`: Show error

### Nvim Tree
- `C-ww`: Toggle file explorer
- `a`: Add file
- `d`: Delete file
- `r`: Rename file
- `x`: Cut file
- `c`: Copy file
- `p`: Paste file
- `R`: Refresh explorer
- `/`: Search file
- `C-t`: Open a file with new tab
