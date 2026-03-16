#include <iostream>
#include <vector>
#include <cmath>
using namespace std;

using Matrix = vector<vector<double>>;
using Vector = vector<double>;

const double EPS = 1e-12; // порог для определения вырожденности

// Ввод матрицы и вектора
void readSystem(Matrix& A, Vector& b, int n) {
    A.assign(n, Vector(n));
    b.assign(n, 0.0);
    cout << "Matrix A:\n";
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < n; ++j)
            cin >> A[i][j];
    cout << "Vector b:\n";
    for (int i = 0; i < n; ++i)
        cin >> b[i];
}

// QR-разложение методом отражений (на месте A и b)
// Возвращает false, если матрица вырождена
bool qrDecomposition(Matrix& A, Vector& b, int n) {
    for (int k = 0; k < n; ++k) {
        // Норма подстолбца
        double norm = 0;
        for (int i = k; i < n; ++i)
            norm += A[i][k] * A[i][k];
        norm = sqrt(norm);

        // Проверка на вырожденность
        if (norm < EPS) return false;

        // Знак для повышения устойчивости
        double s = (A[k][k] > 0) ? -norm : norm;

        // Вектор отражения v
        vector<double> v(n - k);
        for (int i = 0; i < n - k; ++i)
            v[i] = A[k + i][k];
        v[0] -= s;

        // Вычисление tau
        double v2 = 0;
        for (double x : v) v2 += x * x;
        double tau = 2.0 / v2;

        // Применяем преобразование к подматрице A[k:n][k:n]
        for (int j = k; j < n; ++j) {
            double sum = 0;
            for (int i = 0; i < n - k; ++i)
                sum += v[i] * A[k + i][j];
            sum *= tau;
            for (int i = 0; i < n - k; ++i)
                A[k + i][j] -= sum * v[i];
        }

        // Применяем к вектору b (накапливаем Q^T b)
        double sum_b = 0;
        for (int i = 0; i < n - k; ++i)
            sum_b += v[i] * b[k + i];
        sum_b *= tau;
        for (int i = 0; i < n - k; ++i)
            b[k + i] -= sum_b * v[i];
    }
    return true;
}

// Обратная подстановка для верхней треугольной матрицы A (которая теперь R)
Vector solveUpperTriangular(const Matrix& R, const Vector& b, int n) {
    Vector x(n);
    for (int i = n - 1; i >= 0; --i) {
        double s = 0;
        for (int j = i + 1; j < n; ++j)
            s += R[i][j] * x[j];
        x[i] = (b[i] - s) / R[i][i];
    }
    return x;
}

int main() {
    setlocale(LC_ALL, "Russian");
    int n;
    cout << "n = ";
    cin >> n;

    Matrix A;
    Vector b;
    readSystem(A, b, n);

    if (!qrDecomposition(A, b, n)) {
        cout << "nah (matrix is singular)\n";
        return 1;
    }

    Vector x = solveUpperTriangular(A, b, n);

    cout << "Solution:\n";
    for (int i = 0; i < n; ++i)
        cout << "x[" << i << "] = " << x[i] << "\n";

    return 0;
}


// проверка 3x3
n = 3
Matrix A:
2 1 -1
-3 -1 2
-2 1 2
Vector b:
8 -11 -3

Solution:
x[0] = 2
x[1] = 3
x[2] = -1

// проверка 5x5
n = 5
Matrix A:
1 0 0 0 0
0 1 0 0 0
0 0 1 0 0
0 0 0 1 0
0 0 0 0 1
Vector b:
1 2 3 4 5

Solution:
x[0] = 1
x[1] = 2
x[2] = 3
x[3] = 4
x[4] = 5

//Пример работы с особым случаем
n = 3
Matrix A:
1 1 1
2 2 2
3 3 3
Vector b:
1 2 3

nah (matrix is singular)
