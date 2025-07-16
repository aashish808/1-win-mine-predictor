# 1-win-mine-predictor
import streamlit as st
import pandas as pd

# Define board size (5x5)
ROWS = 6
COLS = 5
COLUMNS = ['A', 'B', 'C', 'D', 'E']

# Convert tile like '3A' to (row, col) index
def parse_tile(tile):
    row = int(tile[0])
    col = COLUMNS.index(tile[1].upper())
    return (row, col)

# Convert (row, col) to tile like '3A'
def tile_label(row, col):
    return f"{row}{COLUMNS[col]}"

# Get all adjacent tiles to a given bomb tile
def get_adjacent_tiles(row, col):
    directions = [
        (-1, -1), (-1, 0), (-1, 1),
        (0, -1),          (0, 1),
        (1, -1),  (1, 0), (1, 1)
    ]
    adjacent = []
    for dr, dc in directions:
        nr, nc = row + dr, col + dc
        if 1 <= nr <= ROWS and 0 <= nc < COLS:
            adjacent.append((nr, nc))
    return adjacent

# Main function to predict safe tiles
def predict_safe_tiles(bomb_tiles):
    all_tiles = [(r, c) for r in range(1, ROWS+1) for c in range(COLS)]
    danger_zones = set()
    bomb_positions = set()

    for bomb in bomb_tiles:
        br, bc = parse_tile(bomb)
        bomb_positions.add((br, bc))
        danger_zones.update(get_adjacent_tiles(br, bc))

    unsafe_tiles = bomb_positions.union(danger_zones)
    safe_candidates = [tile for tile in all_tiles if tile not in unsafe_tiles]

    def score(tile):
        tr, tc = tile
        score = 100
        for br, bc in bomb_positions:
            dist = abs(tr - br) + abs(tc - bc)
            score -= max(0, 20 - dist * 5)
        return score

    ranked = sorted(safe_candidates, key=score, reverse=True)
    top_6 = ranked[:6]
    return top_6, bomb_positions, danger_zones

# Streamlit UI
def main():
    st.set_page_config(page_title="1Win Safe Tile Predictor", layout="centered")
    st.title("🎯 1Win Mine Game Safe Tile Predictor")
    st.markdown("""
        ✅ **Enter last game's 3 bomb tile positions** (e.g., `3A`, `5C`, `6E`) below.
        
        🚀 **Get 6 tiles most likely to be safe**, using distance and probability logic.
    """)

    with st.form("predict_form"):
        bomb_input = st.text_input("Bomb tiles (comma-separated):", "3A, 5C, 6E")
        submit = st.form_submit_button("🔍 Predict Safe Tiles")

    if submit:
        try:
            bomb_tiles = [x.strip().upper() for x in bomb_input.split(',') if len(x.strip()) == 2]
            safe_tiles, bombs, danger = predict_safe_tiles(bomb_tiles)
            safe_labels = [tile_label(r, c) for r, c in safe_tiles]

            st.success("🎉 Top 6 Safe Tiles:")
            for tile in safe_labels:
                st.markdown(f"- **{tile}**")

            # Draw grid
            grid = [["" for _ in range(COLS)] for _ in range(ROWS)]
            for r in range(1, ROWS+1):
                for c in range(COLS):
                    label = tile_label(r, c)
                    if (r, c) in bombs:
                        grid[r-1][c] = "💣"
                    elif (r, c) in danger:
                        grid[r-1][c] = "⚠️"
                    elif (r, c) in safe_tiles:
                        grid[r-1][c] = "✅"
                    else:
                        grid[r-1][c] = "▫️"

            df = pd.DataFrame(grid, columns=COLUMNS, index=[str(r) for r in range(1, ROWS+1)])
            st.markdown("---")
            st.subheader("🧠 Visual Grid")
            st.dataframe(df.style.set_properties(**{'text-align': 'center'}), use_container_width=True)

        except Exception as e:
            st.error(f"Error: {e}")

if __name__ == "__main__":
    main()
    
