Toy example

import random
import copy

# -----------------------------
# 1. Toy Data 설정
# -----------------------------

warehouses = ["W1", "W2", "W3"]
items = ["I1", "I2"]
days = [0, 1, 2]  # 3일짜리 toy example

production_cost = {
    "I1": 5,
    "I2": 3
}

# 창고별/일자별/품목별 생산 증가량
stock_increment = {
    "W1": {
        0: {"I1": 10, "I2": 5},
        1: {"I1": 5, "I2": 10},
        2: {"I1": 10, "I2": 5}
    },
    "W2": {
        0: {"I1": 5, "I2": 15},
        1: {"I1": 15, "I2": 5},
        2: {"I1": 5, "I2": 10}
    },
    "W3": {
        0: {"I1": 20, "I2": 5},
        1: {"I1": 5, "I2": 5},
        2: {"I1": 10, "I2": 15}
    }
}

# 주문 데이터
orders = {
    "O1": {
        "day": 0,
        "demand": {"I1": 8, "I2": 4},
        "feasible_warehouses": ["W1", "W2"],
        "transport_cost": {"W1": 10, "W2": 20}
    },
    "O2": {
        "day": 1,
        "demand": {"I1": 12, "I2": 6},
        "feasible_warehouses": ["W1", "W3"],
        "transport_cost": {"W1": 30, "W3": 15}
    },
    "O3": {
        "day": 1,
        "demand": {"I1": 6, "I2": 12},
        "feasible_warehouses": ["W2", "W3"],
        "transport_cost": {"W2": 12, "W3": 25}
    },
    "O4": {
        "day": 2,
        "demand": {"I1": 15, "I2": 5},
        "feasible_warehouses": ["W1", "W2", "W3"],
        "transport_cost": {"W1": 18, "W2": 14, "W3": 20}
    },
    "O5": {
        "day": 2,
        "demand": {"I1": 4, "I2": 16},
        "feasible_warehouses": ["W2", "W3"],
        "transport_cost": {"W2": 25, "W3": 10}
    }
}

# -----------------------------
# 2. 초기 재고 생성
# -----------------------------

def initialize_stock():
    stock = {}

    for w in warehouses:
        stock[w] = {}
        previous = {item: 0 for item in items}

        for d in days:
            stock[w][d] = {}
            for item in items:
                stock[w][d][item] = previous[item] + stock_increment[w][d][item]
                previous[item] = stock[w][d][item]

    return stock

# -----------------------------
# 3. 부족 재고 비용 계산
# -----------------------------

def calculate_stock_cost(stock, order_id, warehouse):
    order = orders[order_id]
    day = order["day"]
    demand = order["demand"]

    shortage_cost = 0

    for item in items:
        available = stock[warehouse][day][item]
        required = demand[item]

        if available < required:
            shortage = required - available
            shortage_cost += shortage * production_cost[item]

    return shortage_cost

# -----------------------------
# 4. 주문 배정 후 재고 업데이트
# -----------------------------

def update_stock(stock, order_id, warehouse):
    order = orders[order_id]
    day = order["day"]
    demand = order["demand"]

    for d in days:
        if d >= day:
            for item in items:
                stock[warehouse][d][item] -= demand[item]

# -----------------------------
# 5. 제약 큰 주문 우선 정렬
# -----------------------------

def constraint_score(order_id):
    order = orders[order_id]

    # 가능한 창고가 적을수록 제약이 큼
    warehouse_constraint = 1 - (len(order["feasible_warehouses"]) / len(warehouses))

    # 수요가 클수록 제약이 큼
    total_demand = sum(order["demand"].values())
    max_possible_demand = 30  # toy example 기준 정규화용 값
    demand_constraint = total_demand / max_possible_demand

    return warehouse_constraint + demand_constraint

# -----------------------------
# 6. Local Search
# -----------------------------

def local_search(stock, order_id, alpha):
    order = orders[order_id]

    best_warehouse = None
    best_cost = float("inf")
    best_transport_cost = None
    best_stock_cost = None

    for w in order["feasible_warehouses"]:
        transport_cost = order["transport_cost"][w]
        stock_cost = calculate_stock_cost(stock, order_id, w)

        # 논문식 핵심:
        # alpha * 운송비 + (1-alpha) * 재고부족비
        total_cost = alpha * transport_cost + (1 - alpha) * stock_cost

        if total_cost < best_cost:
            best_cost = total_cost
            best_warehouse = w
            best_transport_cost = transport_cost
            best_stock_cost = stock_cost

    return best_warehouse, best_cost, best_transport_cost, best_stock_cost

# -----------------------------
# 7. Alpha Tuning
# -----------------------------

def alpha_tuning(alpha, remaining_orders, stock_cost):
    if remaining_orders <= 0:
        return alpha

    step = (1 - alpha) / remaining_orders

    # 재고 부족이 발생하면 재고비용을 더 중요하게 봄
    if stock_cost > 0:
        alpha -= step
    else:
        alpha += step

    # alpha 범위 제한
    alpha = max(0.1, min(0.9, alpha))

    return alpha

# -----------------------------
# 8. GRASP Toy Algorithm
# -----------------------------

def grasp_toy(alpha=0.5):
    stock = initialize_stock()
    assignment = {}
    total_real_cost = 0

    # 제약 큰 주문부터 정렬
    rcl = sorted(orders.keys(), key=constraint_score, reverse=True)

    print("[주문 처리 순서]")
    for o in rcl:
        print(o, "constraint score:", round(constraint_score(o), 3))

    print("\n[배정 과정]")

    while rcl:
        order_id = rcl.pop(0)

        best_w, weighted_cost, transport_cost, stock_cost = local_search(stock, order_id, alpha)

        assignment[order_id] = best_w

        # 실제 비용은 운송비 + 추가 생산비
        real_cost = transport_cost + stock_cost
        total_real_cost += real_cost

        update_stock(stock, order_id, best_w)

        alpha = alpha_tuning(alpha, len(rcl), stock_cost)

        print(
            f"{order_id} -> {best_w} | "
            f"운송비: {transport_cost}, 추가생산비: {stock_cost}, "
            f"실제비용: {real_cost}, alpha: {round(alpha, 3)}"
        )

    return assignment, total_real_cost, stock

# -----------------------------
# 9. 실행
# -----------------------------

assignment, total_cost, final_stock = grasp_toy(alpha=0.5)

print("\n[최종 주문-창고 배정 결과]")
print(assignment)

print("\n[총 비용]")
print(total_cost)

print("\n[최종 재고 상태]")
for w in warehouses:
    print(w, final_stock[w])
