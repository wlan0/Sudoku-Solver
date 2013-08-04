/* Sudoku as a constraint programming problem
 * This code is less than 100 lines long
 * It can solve any 9x9 sudoku
 * Libraries used - IBM iLOG CP OPTIMIZER
 * Sidhartha Mani - sidhartm@andrew.cmu.edu
 */

#include <ilcp/cp.h>

int main(int argc, const char * argv[]) {
    IloEnv env;
    try {
        IloModel model(env);
        int n = 9;
        IloArray<IloIntVarArray> matrix(env,n);
        //All elements in a  row are different
        for(int i=0;i<n;i++)
        {
            matrix[i] = IloIntVarArray(env,n,1,9);
            for(int j=0;j<n;j++)
            {
                model.add(IloAllDiff(env,matrix[i]));
            }
        }
        //All elements in a coloum are different
        for(int i=0;i<n;i++)
        {
            IloIntVarArray window(env);
            for(int j=0;j<n;j++)
            {
                window.add(matrix[j][i]);
            }
            model.add(IloAllDiff(env,window));
        }
        //All elements in each of the 3x3sub-squares are different
        for(int i=0;i<9;i+=3)
        {
            for(int j=0;j<9;j+=3)
            {
                IloIntVarArray window(env);
                for(int k=i;k<i+3;k++)
                {
                    for(int l=j;l<j+3;l++)
                    {
                        window.add(matrix[k][l]);
                    }
                }
                model.add(IloAllDiff(env,window));
            }
        }
        //Already provided numbers
        int PreSolve[][9] = {9,5,0,3,7,0,0,0,0,
                            0,0,0,0,0,0,0,0,0,
                            0,0,4,0,8,0,5,7,0,
                            3,9,0,0,0,0,0,0,4,
                            0,0,5,0,4,0,8,0,0,
                            8,0,0,0,0,0,0,1,7,
                            0,6,8,0,3,0,7,0,0,
                            0,0,0,0,0,0,0,0,0,
                            0,0,0,0,1,8,0,2,9};
        //Add presolved amtrix
        for(int i=0;i<n;i++)
        {
            for(int j=0;j<n;j++)
            {
                if(PreSolve[i][j] != 0)
                model.add(matrix[i][j] == PreSolve[i][j]);
            }
        }
        //Solve
        IloCP cp(model);
        if(cp.solve())
        {
            for(IloInt i = 0;i<n;i++)
            {
                for(int j=0;j<n;j++)
                {
                    cp.out() << cp.getValue(matrix[i][j]) << "\t";
                }
                cp.out()<<std::endl;
            }
        }
        
    }
    catch (IloException & ex) {
        env.out() << "Exception Caught : " << ex << std::endl;
    }
}
