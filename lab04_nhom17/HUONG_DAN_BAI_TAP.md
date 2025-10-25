# HƯỚNG DẪN BÀI TẬP: DOTS AND BOXES

## Tổng quan

Bài tập này implement các thuật toán AI để chơi trò chơi Dots and Boxes, từ cơ bản (Random) đến nâng cao (Minimax, Heuristic, Monte Carlo).

## Cấu trúc file

- `assignment_dots_and_boxes.ipynb` - Notebook chính chứa toàn bộ code và documentation

## Các Task đã hoàn thành

### ✅ Task 1: Định nghĩa bài toán tìm kiếm (10 điểm)

**Nội dung:**
- Định nghĩa 5 thành phần: Initial state, Actions, Transition model, Terminal test, Utility
- Phân tích kích thước không gian trạng thái
- Ước tính kích thước game tree

**Kết quả:**
- Không gian trạng thái: 2^L với L = số đường kẻ (2mn - m - n)
- Game tree: Phức tạp hơn do luật "đi tiếp khi hoàn thành box"
- Board 3×3: 12 lines, 2^12 = 4096 states
- Board 5×5: 40 lines, 2^40 = ~1 trillion states

### ✅ Task 2: Game Environment & Random Agent (30 điểm)

**Implementation:**

1. **Board Representation:**
```python
board = {
    'size': (4, 4),      # Số hàng và cột điểm
    'lines': {},         # Dict các đường kẻ đã vẽ
    'boxes': {}          # Dict boxes đã hoàn thành
}
```

2. **Helper Functions:**
- `actions(board)` - Lấy danh sách nước đi hợp lệ
- `result(board, action, player)` - Apply action, trả về board mới và next player
- `terminal(board)` - Kiểm tra game kết thúc
- `utility(board, player)` - Tính điểm
- `display_board(board)` - Visualization bằng ASCII art

3. **Random Agent:**
- Chọn ngẫu nhiên từ available actions
- Baseline để so sánh các agents khác

**Thực nghiệm:**
- 1000 games giữa 2 random players
- Kết quả: Player đi trước thắng ~52-55% (có lợi thế nhỏ)

### ✅ Task 3: Minimax với Alpha-Beta Pruning (30 điểm)

**Implementation:**

```python
def minimax_alpha_beta(board, player, alpha, beta, maximizing):
    # Base cases
    if terminal(board):
        return utility(board, player), None
    
    # Recursive cases với pruning
    # Đặc biệt: Xử lý rule "đi tiếp khi hoàn thành box"
```

**Các cải tiến:**

1. **Move Ordering:**
   - Completing moves (hoàn thành box) → ưu tiên cao nhất
   - Safe moves (không tạo box 3 cạnh) → ưu tiên trung bình
   - Risky moves (tạo box 3 cạnh) → ưu tiên thấp nhất
   - **Kết quả:** Giảm 30-50% nodes cần explore

2. **Symmetry Reduction:**
   - Loại bỏ moves đối xứng trong opening
   - Giảm ~50-75% search space cho first move

3. **Opening Book:**
   - Hardcode best first moves cho các board sizes
   - Tránh phải search toàn bộ tree từ empty board

**Performance:**
- Board 3×3: ~1-5 giây cho first move
- Board 4×4: ~30-60 giây (có thể quá chậm)
- Board lớn hơn: Không khả thi với full minimax

### ✅ Task 4: Heuristic Alpha-Beta (30 điểm)

**Heuristic Evaluation Function:**

```python
def heuristic_evaluation(board, player):
    score = 0
    score += (completed_boxes) × 100           # Quan trọng nhất
    score -= (almost_complete_boxes) × 50      # Nguy hiểm
    score += (half_complete_boxes) × 10        # Tiềm năng
    score += (neutral_boxes) × 2               # Linh hoạt
    score += bonus_if_leading × 30             # Dẫn trước
    return score
```

**Depth-Limited Search:**
- Giới hạn độ sâu search
- Sử dụng heuristic thay vì terminal utility khi đạt max_depth
- Trade-off: Optimal vs Speed

**Chiến lược chọn Depth:**
- Board 3×3: Depth 4-6
- Board 4×4: Depth 3-4
- Board 5×5: Depth 2-3
- Adaptive: Early game dùng depth thấp, endgame dùng depth cao

**Performance:**
- Board 4×4 với depth 4: ~2-5 giây
- Board 5×5 với depth 4: ~5-10 giây
- Khả thi cho board lớn

### ✅ Advanced Task: Pure Monte Carlo Search (10 điểm bonus)

**Implementation:**

```python
def pure_monte_carlo_search(board, player, num_simulations):
    # Với mỗi possible move:
    #   1. Apply move
    #   2. Chạy N random simulations đến terminal
    #   3. Đếm số thắng/thua
    # Chọn move có win rate cao nhất
```

**Ưu điểm:**
- Không cần heuristic evaluation
- Không cần domain knowledge
- Scale tốt với board lớn
- Anytime algorithm

**Nhược điểm:**
- Không optimal
- Cần nhiều simulations (100-1000+)
- Chậm hơn heuristic search với board nhỏ

**Use cases:**
- Board lớn (5×5+)
- Khi thiếu domain knowledge
- Games có random elements

**Best First Move Analysis:**
- Board 5×5: 40 possible first moves
- Symmetry reduction: ~10 unique moves
- Monte Carlo với 200 sims/move
- Kết luận: Moves gần center tốt nhất

## So sánh các thuật toán

| Thuật toán | Optimal? | Speed | Max Board | Complexity |
|------------|----------|-------|-----------|------------|
| Random | ❌ | ⚡⚡⚡ | Unlimited | O(1) |
| Minimax Full | ✅ | 🐌 | 3×3, 3×4 | O(b^d) |
| Minimax + Ordering | ✅ | 🐌⚡ | 3×3, 3×4 | O(b^d) optimized |
| Heuristic (d=4) | ⚠️ | ⚡⚡ | 5×5 | O(b^d) with cutoff |
| Monte Carlo | ❌ | ⚡ | 7×7+ | O(n × m) |

**Chú thích:**
- ⚡ = Nhanh
- 🐌 = Chậm
- ✅ = Optimal
- ⚠️ = Near-optimal
- ❌ = Not optimal

## Kết quả Tournament

**Minimax vs Random:**
- Minimax thắng: ~100% (trên board 3×3)

**Heuristic Depth-4 vs Depth-2:**
- Depth-4 thắng: ~80-90%

**Monte Carlo vs Random:**
- Monte Carlo thắng: ~90-95%

**Minimax vs Monte Carlo:**
- Minimax thắng: ~70-80% (trên board 3×3)
- Monte Carlo cạnh tranh tốt hơn trên board lớn

## Cải tiến có thể thực hiện

1. **Monte Carlo Tree Search (MCTS):**
   - Kết hợp tree search và simulation
   - UCB1 để balance exploration/exploitation
   - Mạnh hơn Pure MC

2. **Transposition Table:**
   - Cache các states đã evaluate
   - Tránh re-compute

3. **Iterative Deepening:**
   - Tăng depth dần
   - Có answer bất cứ lúc nào
   - Time management tốt hơn

4. **Neural Network:**
   - Học evaluation function từ data
   - AlphaZero-style approach

5. **Advanced Strategies:**
   - Chain detection and analysis
   - Control theory (Berlekamp)
   - Endgame database

## Cách chạy code

1. Mở file `assignment_dots_and_boxes.ipynb` trong VS Code hoặc Jupyter
2. Chọn Python kernel (Python 3.7+)
3. Run All Cells (hoặc run từng cell)

**Lưu ý:**
- Không cần install packages (chỉ dùng built-in libraries)
- Một số cells chạy lâu (1-3 phút)
- Có thể giảm num_games/num_simulations để test nhanh

## Thời gian thực hiện

- Task 1: ~1 giờ
- Task 2: ~2-3 giờ
- Task 3: ~3-4 giờ
- Task 4: ~2-3 giờ
- Advanced: ~2 giờ
- **Tổng:** ~10-13 giờ

## Tài liệu tham khảo

- Russell & Norvig - Artificial Intelligence: A Modern Approach (Chapter 5)
- [Dots and Boxes - Wikipedia](https://en.wikipedia.org/wiki/Dots_and_Boxes)
- [Tic-Tac-Toe examples from CS7320-AI](https://github.com/mhahsler/CS7320-AI)
- Berlekamp - "The Dots and Boxes Game" (1974)

## Liên hệ

Nếu có câu hỏi về implementation, vui lòng tạo issue hoặc liên hệ qua email.

---

**Lưu ý:** Code này được viết cho mục đích học tập. Có thể optimize thêm cho production use.
