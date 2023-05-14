# nvim-coverage

Displays coverage information in the sign column.

![markers](https://user-images.githubusercontent.com/542263/159128715-32e6eddf-5f9f-4853-9e2b-abd66bbf01d4.png)

Displays a coverage summary report in a pop-up window.

![summary](https://user-images.githubusercontent.com/542263/159128732-8189b89d-4f71-4a34-8c6a-176e40fcd48d.png)

Currently supports:

- C/C++ (lcov)
- C# (lcov - see wiki for details)
- Dart (lcov)
- Go (coverprofile)
- Javascript/Typescript (lcov): [jest](https://jestjs.io/docs/getting-started)
- Julia (lcov): [Pkg.jl](https://pkgdocs.julialang.org/v1/)
- Python (json): [coverage.py](https://coverage.readthedocs.io/en/6.3.2/index.html)
- Ruby (json): [SimpleCov](https://github.com/simplecov-ruby/simplecov)
- Rust (json): [grcov](https://github.com/mozilla/grcov#usage)
- PHP (cobertura)
- Anything that generates lcov files

Branch (partial) coverage support:

| Language              | Supported              |
| --------------------- | ---------------------- |
| C/C++                 | :x: |
| C#                    | :x: |
| Dart                  | :heavy_check_mark: (untested) |
| Go                    | :x: |
| Javascript/Typescript | :heavy_check_mark: |
| Julia                 | :heavy_check_mark: (untested) |
| Python                | :heavy_check_mark: |
| Ruby                  | :x: |
| Rust                  | :x: |
| PHP                   | :x: |

*Note:* This plugin does not run tests. It justs loads/displays a coverage report generated by a test suite.
To run tests from neovim with coverage enabled, try one of these plugins:

* [neotest](https://github.com/nvim-neotest/neotest)
* [vim-test](https://github.com/vim-test/vim-test)

## Installation

This plugin depends on [plenary](https://github.com/nvim-lua/plenary.nvim) and optionally on the
[lua-xmlreader](https://luarocks.org/modules/luarocks/lua-xmlreader) luarock to parse the cobertura format.

Using vim-plug (not including the luarock dependency):
```vim
Plug 'nvim-lua/plenary.nvim'
Plug 'andythigpen/nvim-coverage'
```

The following lua is required to configure the plugin after installation.
```lua
require("coverage").setup()
```

Using packer:
```lua
use({
  "andythigpen/nvim-coverage",
  requires = "nvim-lua/plenary.nvim",
  -- Optional: needed for PHP when using the cobertura parser
  rocks = { 'lua-xmlreader' },
  config = function()
    require("coverage").setup()
  end,
})
```

## Configuration

See [docs](https://github.com/andythigpen/nvim-coverage/blob/main/doc/nvim-coverage.txt) for more info.

Example:

```lua
require("coverage").setup({
	commands = true, -- create commands
	highlights = {
		-- customize highlight groups created by the plugin
		covered = { fg = "#C3E88D" },   -- supports style, fg, bg, sp (see :h highlight-gui)
		uncovered = { fg = "#F07178" },
	},
	signs = {
		-- use your own highlight groups or text markers
		covered = { hl = "CoverageCovered", text = "▎" },
		uncovered = { hl = "CoverageUncovered", text = "▎" },
	},
	summary = {
		-- customize the summary pop-up
		min_coverage = 80.0,      -- minimum coverage threshold (used for highlighting)
	},
	lang = {
		-- customize language specific settings
	},
})
```

## Extending to other languages

1. Create a new lua module matching the pattern `coverage.languages.<filetype>` where `<filetype>` matches the vim filetype for the coverage language (ex. python).
2. Implement the required methods listed below.

Required methods:
```lua
local M = {}

--- Loads a coverage report.
-- This method should perform whatever steps are necessary to generate a coverage report.
-- The coverage report results should passed to the callback, which will be cached by the plugin.
-- @param callback called with results of the coverage report
M.load = function(callback)
  -- TODO: callback(results)
end

--- Returns a list of signs that will be placed in buffers.
-- This method should use the coverage data (previously generated via the load method) to 
-- return a list of signs.
-- @return list of signs
M.sign_list = function(data)
  -- TODO: generate a list of signs using:
  -- require("coverage.signs").new_covered(bufnr, linenr)
  -- require("coverage.signs").new_uncovered(bufnr, linenr)
end

--- Returns a summary report.
-- @return summary report
M.summary = function(data)
  -- TODO: generate a summary report in the format
  return {
    files = {
      { -- all fields, except filename, are optional - the report will be blank if the field is nil
        filename = fname,            -- filename displayed in the report
        statements = statements,     -- number of total statements in the file
        missing = missing,           -- number of lines missing coverage (uncovered) in the file
        excluded = excluded,         -- number of lines excluded from coverage reporting in the file
        branches = branches,         -- number of total branches in the file
        partial = partial_branches,  -- number of branches that are partially covered in the file
        coverage = coverage,         -- coverage percentage (float) for this file
      }
    },
    totals = { -- optional
      statements = total_statements,     -- number of total statements in the report
      missing = total_missing,           -- number of lines missing coverage (uncovered) in the report
      excluded = total_excluded,         -- number of lines excluded from coverage reporting in the report
      branches = total_branches,         -- number of total branches in the report
      partial = total_partial_branches,  -- number of branches that are partially covered in the report
      coverage = total_coverage,         -- coverage percentage to display in the report
    }
  }
end

return M
```
