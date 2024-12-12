# neovim-tool

分享一些有用的工具脚本，有可能不是源创的

### 根据缩进跳转
```
local GoToNextIndent = function(inc)
  -- Get the cursor current position
  local currentPos = fn.getpos('.')
  local currentLine = currentPos[2]
  local origCol = currentPos[3]
  local origIndent = fn.indent('.')
  local ok = false

  -- print(("%s-%s"):format(origCol, origIndent))
  -- Look for a line with the same indent level whithout going out of the buffer
  local begin_ = -1
  local end_ = fn.line('$') + 1
  local up_level = origCol <= origIndent
  local down_level = origCol > origIndent + 1

  while (not ok and currentLine ~= end_ and currentLine ~= begin_) do
    currentLine = currentLine + inc
    local line_noempty = fn.len(fn.getline(currentLine)) > 0
    if (line_noempty) then
      if (up_level) then
        ok = fn.indent(currentLine) < origIndent  -- up level
      elseif (down_level) then
        ok = fn.indent(currentLine) > origIndent  -- down level
      else
        ok = fn.indent(currentLine) == origIndent -- same level
      end
    end
  end

  -- If a line is found go to this line
  if (ok) then
    currentPos[2] = currentLine
    if (up_level) then
      currentPos[3] = fn.indent(currentLine)
    elseif (down_level) then
      currentPos[3] = fn.indent(currentLine) + 2
    else
      currentPos[3] = fn.indent(currentLine) + 1
    end
    fn.setpos('.', currentPos)
  end
end
map("n", "g]", "", { callback = function() return GoToNextIndent(1) end })
map("n", "g[", "", { callback = function() return GoToNextIndent(-1) end })
map("v", "g]", "", { callback = function() return GoToNextIndent(1) end })
map("v", "g[", "", { callback = function() return GoToNextIndent(-1) end })

```

### 删除其他只剩下选中的
```
local ClearAllButMatches = function(opts)
  local ls, le = opts.line1, opts.line2
  local is_whole_file = (ls == 1 and le == vim.fn.line("$"))
  local old_c = vim.fn.getreg("c")

  vim.fn.setreg("c", "")

  local cmd_add_reg_c = string.format([[%s,%ssub//\=setreg("C", submatch(0), "l")/g]], ls, le)
  local cmd_del_selection = string.format([[%s,%sdelete _]], ls, le)

  vim.api.nvim_exec2(cmd_add_reg_c, {})

  if is_whole_file then
    vim.api.nvim_exec2("$delete _", {})
  else
    vim.api.nvim_exec2(cmd_del_selection, {})
  end

  vim.api.nvim_exec2("put! c", {})

  vim.fn.setreg("c", old_c)
end
vim.api.nvim_create_user_command("Cabm", ClearAllButMatches, { range = true })
```
