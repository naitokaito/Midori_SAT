# Midori_SAT
軽量暗号であるMidoriを評価するためにSATを用いた。その際におけるSATのコード


from pysat.solvers import Solver
from pysat.formula import CNF
from pysat.card import *
from itertools import chain
import subprocess
import numpy as np

    

def is_check_result(cnf_path,log_path):
    with open(log_path,'r') as f:
        result = f.read()  
        if 's UNSATISFIABLE' in result:
            cmd = f'rm {cnf_path}'
            subprocess.run(cmd,shell=True)
            cmd = f'rm {log_path}'
            subprocess.run(cmd,shell=True)
            return False
        elif 's SATISFIABLE' in result:
            cmd = f'rm {cnf_path}'
            subprocess.run(cmd,shell=True)
            return True
        
def midori_sb0_sbox(cnf,in0,in1,in2,in3,out0,out1,out2,out3,prob0,prob1,prob2):
    cnf.append([-in2,prob0])
    cnf.append([-out2,prob0])
    cnf.append([-prob0,prob1])
    cnf.append([prob0,-prob2])
    cnf.append([in1,-in2,in3,-prob2])
    cnf.append([in1,-in3,out2,prob2])
    cnf.append([in2,-in3,out2,prob2])
    cnf.append([-in1,in3,out2,prob2])
    cnf.append([in2,out1,-out3,prob2])
    cnf.append([in2,-out1,out3,prob2])
    cnf.append([-in0,out2,out3,prob2])
    cnf.append([in1,in2,in3,-out1,-out3])
    cnf.append([in0,-in2,in3,-out2,out3])
    cnf.append([-in1,-in2,-in3,out2,out3])
    cnf.append([in0,in1,in2,in3,-prob1])
    cnf.append([out0,out1,out2,out3,-prob1])
    cnf.append([-in1,-in3,out0,out1,prob2])
    cnf.append([in0,in1,-out1,-out3,prob2])
    cnf.append([in1,-in3,-out1,-out3,prob2])
    cnf.append([-in1,in3,-out1,-out3,prob2])
    cnf.append([-in0,in3,out1,-out3,prob2])
    cnf.append([-in1,-out0,out1,-out3,prob2])
    cnf.append([in1,-in3,-out0,out3,prob2])
    cnf.append([-in3,-out0,-out1,out3,prob2])
    cnf.append([-in1,-in2,-out0,out1,out2,-out3])
    cnf.append([in1,in2,in3,out1,out3,-prob1])
    cnf.append([-in2,in3,out0,-out1,out2,-prob2])
    cnf.append([-in2,-in3,out0,out2,-out3,-prob2])
    cnf.append([-in0,in1,-in3,-out2,out3,-prob2])
    cnf.append([in0,in3,-out0,out1,out3,prob2])
    cnf.append([in0,in3,-out0,out1,out2,-out3,-prob2])
    cnf.append([-in0,in1,in2,-in3,out0,out3,-prob2])
    cnf.append([in0,in1,-out0,-out1,out2,out3,-prob2])
    cnf.append([-in1,-in3,out1,out3,-prob0,-prob1,-prob2])
    cnf.append([-in0,-in2,out0,-out2,-prob0,-prob1,prob2])
    cnf.append([-in0,-in1,in3,-out0,out1,-out2,-prob0,-prob1])
    cnf.append([-in0,in1,-in2,-out0,-out1,out3,-prob0,-prob1])
    cnf.append([in0,in2,-in3,out1,-out2,-prob0,-prob1,-prob2])
    cnf.append([in2,out0,-out1,-out2,-out3,-prob0,-prob1,-prob2])
    cnf.append([in0,-in1,in2,-out2,out3,-prob0,-prob1,-prob2])
    cnf.append([in0,-in1,-in2,-in3,-out2,-prob0,-prob1,prob2])
    cnf.append([in0,in1,-in2,out1,-out2,-prob0,-prob1,prob2])
    cnf.append([-in0,-in1,in2,in3,out0,out1,-prob0,-prob1,-prob2])
    cnf.append([-in2,in3,out1,-out2,-prob2])
    cnf.append([in1,-in2,-out2,out3,-prob2])
    cnf.append([-in1,-in3,-out1,-out3,-prob2])
    cnf.append([in2,-out0,-out1,-out3])
    cnf.append([-in0,-in3,out0,out1,out2])
    cnf.append([in1,-in2,out1,-out2,-prob2])
    cnf.append([-in0,-in1,out0,out2,out3])
    cnf.append([-in2,in3,-out2,out3,-prob2])
    cnf.append([in0,in1,in2,-out0,-out3])
    cnf.append([in0,in2,in3,-out0,-out1])
    cnf.append([-in0,-in1,-in3,out2])
    
    #mixcolumnのCNFをかく、SATから
def Mix_Column(cnf,in0,in1,in2,out0):
    cnf.append([-in0,in1,in2,out0])
    cnf.append([in0,-in1,in2,out0])
    cnf.append([in0,in1,-in2,out0])
    cnf.append([-in0,-in1,-in2,out0])
    cnf.append([in0,in1,in2,-out0])
    cnf.append([-in0,-in1,in2,-out0])
    cnf.append([-in0,in1,-in2,-out0])
    cnf.append([in0,-in1,-in2,-out0])
    return cnf


def main(round,weight):

    bit_length = 64#bitの長さ
    prob_length = 48#3bit*16個
    #UNSATやったらweightを加える。SATやったらweightを加えない。
    
    CNFPATH = './cnf_file.cnf'
    LOGPATH = './log_file.log'
    
    cnf = []
    sin = []
    sout = []
    prob = []
    var_count = 1
    sin += [[i * bit_length + j + var_count for j in range(bit_length)] for i in range(round+1)]
    var_count += bit_length * (round +1)
    sout += [[i * bit_length + j + var_count for j in range(bit_length)] for i in range(round)]
    var_count += bit_length * round
    prob += [[i * prob_length + j + var_count for j in range(prob_length)] for i in range(round)]
    var_count += prob_length * round
    #print(sout)

    cnf.append(sin[0])
    for r in range(round):
        for i in range(16):
            midori_sb0_sbox(cnf,sin[r][4*i],sin[r][4*i+1],sin[r][4*i+2],sin[r][4*i+3],sout[r][4*i],sout[r][4*i+1],sout[r][4*i+2],sout[r][4*i+3],prob[r][3*i],prob[r][3*i+1],prob[r][3*i+2])
            #ここまで書き換え

        #ここにシャッフルセルimport numpy as np
        shuffle_l = [0, 10, 5, 15, 14, 4, 11, 1, 9, 3, 12, 6, 7, 13, 2, 8]
        sin_list = []
        sout_numpy = np.array(sout[r])#sout_numpy,sin_numpyは上書きされる。つまり新しく箱を作る必要はない
        sin_numpy = np.array(sin[r])
        sout_numpy = sout_numpy.reshape(16,4)#shufflecelは一次元だからrehsapeできる。
        sin_numpy = sin_numpy.reshape(16,4)
        #print(sin_numpy)
                
        for k in range(16):
            sin_numpy[k] = sout_numpy[shuffle_l[k]]
#16行4列をリストに戻す
        for column in range(16):
            for row in range(4):
                sin_list.append(sin_numpy[column][row])
        

        #ここにMixClumns,sinを最初のmainの中に戻すから
        for i in range(4):
            for j in range(4):
                Mix_Column(cnf, sin_list[16 * i + (j + 4)], sin_list[16 * i + (j + 8)], sin_list[16 * i + (j + 12)], sin[r+1][16 * i + j])#最後のやつだけどこに入れるかrで指定する必要がある。
                Mix_Column(cnf, sin_list[16 * i + j], sin_list[16 * i + (j + 8)], sin_list[16 * i + (j + 12)], sin[r+1][16 * i + (j + 4)])
                Mix_Column(cnf, sin_list[16 * i + j], sin_list[16 * i + (j + 4)], sin_list[16 * i + (j + 12)], sin[r+1][16 * i + (j + 8)])
                Mix_Column(cnf, sin_list[16 * i + j], sin_list[16 * i + (j + 4)], sin_list[16 * i + (j + 8)], sin[r+1][16 * i + (j + 12)])




    cnf.extend(CardEnc.atmost(lits=list(chain.from_iterable(prob)),bound=weight,encoding=6,top_id=var_count))
    cnf = CNF(from_clauses=cnf)
    cnf.to_file(CNFPATH)
    
    SOLVERPATH = './kissat-rel-3.1.1/build/kissat'
    cmd = f'{SOLVERPATH} {CNFPATH} > {LOGPATH}'#なんか分からんけど--relaxedを入れた。直ったから、やっぱいらんかった
    subprocess.run(cmd,shell=True)
    
    is_result = is_check_result(CNFPATH,LOGPATH) 
    return is_result
    

if __name__ == '__main__':
    weight = 1
    for round in range(1,11):
        while(1):
            if main(round,weight) ==True:
                print(round,weight,"SAT")
                weight += 1
                break
            else:
                print(round,weight,"UNSAT")
                weight += 1


    
